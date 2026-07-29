# Docker container daemon architecture


- A container engine is a software tool that accepts and processes requests from users to create a container; it can be seen as a sort of orchestrator. 
- A container runtime is a lower-level piece of software used by container engines to run containers on the host, managing isolation, storage, networking, and so on.

The Docker container engine consists of three fundamental pillars:
- Docker daemon
- Docker REST API
- Docker CLI

![](images/B31467_2_1.png)

- Once a **Docker** **daemon** is running, you can interact with it through a Docker **client** or a **remote API**. 
- The Docker daemon is responsible for many local container activities, as well as interacting with external registries to pull or push **container** **images**.
- The Docker daemon is the most critical piece of the architecture, and it should always be up and running; otherwise, your beloved containers will not survive for long!

## The Docker daemon 

The Docker daemon is the background process that is responsible for the following:
- Listening for Docker API requests
- Handling, managing, and checking for running containers
- Managing Docker images, networks, and storage volumes
- Interacting with external/remote container image registries

- All these actions should be instructed to the daemon through a client or by calling its API.

## Interacting with the Docker daemon 

- The Docker daemon can be contacted through the socket of a process, usually available in the filesystem of the host machine: `/var/run/docker.sock`.

- Depending on the Linux distribution of your choice, you may need to set the right permissions for your non-root users to be able to interact with the Docker daemon or simply add your non-privileged users to the `docker` group.

Permissions set for the Docker daemon in a Fedora 40 operating system:
```
ls -la /var/run/docker.sock srw-rw----. 1 root docker 0 Aug 25 12:48 /var/run/docker.sock
```

- There is no other kind of security or authentication for a Docker daemon enabled by default
- Be careful not to publicly expose the daemon to untrusted networks
- Access to the daemon's socket would be equivalent to passwordless root access on the system.

## The Docker REST API 

- Can do every kind of activity you can perform through the command-line tool.
	- List containers
	- Create a container
	- Inspect a container
	- Get container logs
	- Export a container
	- Start or stop a container
	- Kill a container
	- Rename a container
	- Pause a container

Use the Linux command-line tool `curl` to make a **HyperText Transfer Protocol** (**HTTP**) request to get details about any container image already stored in the daemon\'s local cache:

```
curl --unix-socket /var/run/docker.sock \ http://localhost/v1.41/images/json | jq [
 { "Containers": -1, "Created": 1724036893, "Id": "sha256:a95dd4643de6c46ee41ea5d8b7bb4049ce82453e8ec9c8238b14e729219541fe", "Labels": { "architecture": "x86_64", "build-date": "2024-08-19T03:00:40", "com.redhat.component": "ubi9-container", "com.redhat.license_terms": "https://www.redhat.com/en/about/red-hat-end-user-license-agreements#UBI", "description": "The Universal Base Image is designed and engineered to be the base layer for all of your containerized applications, middleware and utilities. This base image is freely redistributable, but Red Hat only supports Red Hat technologies through subscriptions for Red Hat products. This image is maintained by Red Hat and updated regularly.", "distribution-scope": "public", "io.buildah.version": "1.29.0", "io.k8s.description": "The Universal Base Image is designed and engineered to be the base layer for all of your containerized applications, middleware and utilities. This base image is freely redistributable, but Red Hat only supports Red Hat technologies through subscriptions for Red Hat products. This image is maintained by Red Hat and updated regularly.", "io.k8s.display-name": "Red Hat Universal Base Image 9", "io.openshift.expose-services": "", "io.openshift.tags": "base rhel9", "maintainer": "Red Hat, Inc.", "name": "ubi9", "release": "1181.1724035907", "summary": "Provides the latest release of Red Hat Universal Base Image 9.", "url": "https://access.redhat.com/containers/#/registry.access.redhat.com/ubi9/images/9.4-1181.1724035907", "vcs-ref": "e309397d02fc53f7fa99db1371b8700eb49f268f", "vcs-type": "git", "vendor": "Red Hat, Inc.", "version": "9.4" }, "ParentId": "", "RepoDigests": [ "registry.access.redhat.com/ubi9/ubi@sha256:9e6a89ab2a9224712391c77fab2ab01009e387aff42854826427aaf18b98b1ff" ], "RepoTags": [ "registry.access.redhat.com/ubi9/ubi:9.4-1181.1724035907" ], "SharedSize": -1, "Size": 212442936, "VirtualSize": 212442936 } ]
```

- Output is in JSON format
- Very detailed with multiple metadata information.

## Docker client commands 

- Enable any system administrator or Docker user to instruct and control the daemon and its containers. 
- Common commands:
	- `build`: Build an image from a Dockerfile
	- `cp`: Copy files/folders between a container and the local filesystem
	- `exec`: Run a command in a running container
	- `images`: List images
	- `inspect`: Return low-level information on Docker objects
	- `kill`: Kill one or more running containers
	- `load`: Load an image from a tar archive or STDIN
	- `login`: Log in to a Docker registry
	- `logs`: Fetch the logs of a container
	- `ps`: List running containers
	- `pull`: Pull an image or a repository from a registry
	- `push`: Push an image or a repository to a registry
	- `restart`: Restart one or more containers
	- `rm`: Remove one or more containers
	- `rmi`: Remove one or more images
	- `run`: Run a command in a new container
	- `save`: Save one or more images to a TAR archive (streamed to stdout by default)
	- `create`: Create a new container from a specified image without starting it.
	- `start`: Start one or more stopped containers
	- `stop`: Stop one or more running containers
	- `tag`: Create a `TARGET_IMAGE` tag that refers to `SOURCE_IMAGE`

- Once you launch the Docker client with one of these commands and its respective options, the client will contact the Docker daemon, where it\'ll instruct it on what is needed and which action must be performed. 
- The daemon needs to be up and running.

## Docker images 

- Format introduced by Docker for managing binary data and metadata as a template for container creation.
- Packages for shipping and transferring runtimes, libraries, and all the stuff needed for a given process to be up and running.
- Adheres to the **OCI Image Format Specification**.
	- A list of layers
	- Creation date
	- Operating system
	- CPU architecture
	- Configuration parameters for use within a container runtime
- Content (binaries, libraries, filesystem data) is organized in layers. 
- A layer is just a set of filesystem changes that does not contain any environment variables or default arguments for a given command. 
	- Data is stored in the **image** **manifest** that owns the configuration parameters.
- Composed together using image metadata and merged into a single filesystem view. 
- Most common approach is by using union filesystems 
	- Combining two filesystems and providing a unique, *squashed* view. 
- When a container is executed, a new, *read/write* ephemeral layer is created on top of the image, which will be lost after the container is destroyed.

## Docker registries 

- A repository of Docker container images that holds the metadata and the layers of container images.
- Docker daemon acts as a client to a Docker registry through an HTTP API, pushing and pulling container images depending on the action that the Docker client instructs.
- Facilitate the use of containers on many independent machines. 
- Machines can be configured to pull container images from a registry if they are not present in the Docker daemon local cache. 
- The default registry that is preconfigured in Docker daemon settings is **Docker** **Hub**, a **Software-as-a-Service** container registry hosted by the Docker company in the cloud. 
- **Quay.io**
	- A Software-as-a-Service container registry hosted by Red Hat.
- On-premises Docker registry
	- Which can be created through a container on a machine running the Docker daemon with just one command:

``` $ docker run -d -p 5000:5000 --restart=always --name registry registry:2 ```

## What does a running Docker architecture look like? 

```
# systemctl status docker ● docker.service
- Docker Application Container Engine Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; preset: disabled) Drop-In: /usr/lib/systemd/system/service.d └─10-timeout-abort.conf Active: active (running) since Tue 2024-09-03 20:01:46 UTC; 4min 0s ago TriggeredBy: ● docker.socket Docs: https://docs.docker.com Main PID: 3970 (dockerd) Tasks: 11 Memory: 171.4M (peak: 237.3M swap: 4.0K swap peak: 4.0K) CPU: 4.062s CGroup: /system.slice/docker.service └─3970 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock 0
```

**containerd**
- Docker daemon companion.

![](images/B31467_2_2.png)

- Decouples the container management (kernel interaction included) from the Docker daemon\.
- Adheres to the OCI standard using `runc` as the container runtime.

Check the status of containerd in our preconfigured operating system:

``` 0# systemctl status containerd ● containerd.service
- containerd container runtime Loaded: loaded (/usr/lib/systemd/system/containerd.service; disabled; preset: disabled) Drop-In: /usr/lib/systemd/system/service.d └─10-timeout-abort.conf Active: active (running) since Tue 2024-09-03 20:01:45 UTC; 8min ago Docs: https://containerd.io Process: 3960 ExecStartPre=/sbin/modprobe overlay (code=exited, status=0/SUCCESS) Main PID: 3962 (containerd) Tasks: 18 Memory: 93.4M (peak: 111.9M swap: 3.0M swap peak: 5.6M) CPU: 359ms CGroup: /system.slice/containerd.service ├─3962 /usr/bin/containerd └─4231 /usr/bin/containerd-shim-runc-v2 -namespace moby -id a478c626278a1aa388732501c14fe9f4b4dfab70ad8d87266cdcc770adf3e93b -address /run/containerd/containerd.sock ```
```

The service is up and running, and it has started one child process: `/usr/bin/containerd-shim-runc-v2`.

## containerd architecture 

- Composed of several components that are organized in subsystems. 
- Components that link different subsystems are also referred to as modules in the containerd architecture:

![](images/B31467_2_3.png)

The two main subsystems:
- The bundle service that extracts bundles from disk images
- The runtime service that executes the bundles, creating the runtime containers

The main modules that make the architecture fully functional are the following:
- The `Executor` module, which implements the container runtime that is represented in the preceding architecture as the **Runtimes** block.
- The `Supervisor` module, which monitors and reports the container state that is part of the **Containers** block in the preceding architecture
- The `Snapshot` module, which manages filesystem snapshots.
- The `Events` module, which collects and consumes events.
- The `Metrics` module, which exports several metrics via the metrics API.

The steps needed by containerd to place a container in a running state are summed up as follows:
1. Pull metadata and content through a **distribution** **controller**.
2. Use the **bundle** **controller** to unpack the retrieved data, creating snapshots that will compose bundles
3. Execute the container through the bundle just created through the **runtime** **controller**:

![](images/B31467_2_4.png)

# Podman daemonless architecture 

Podman (short for *POD MANager*)
- Daemonless container engine 
- Enables users to manage containers, images, and their related resources, such as storage volumes or network resources. 
- No service to start after the installation is complete.
- Podman binary acts both as a **command-line interface** (**CLI**) and as a container engine that orchestrates the container runtime execution.

## Podman commands and REST API 

The Podman CLI provides a growing set of commands: https://docs.podman.io/en/latest/Commands.html.

Commonly used commands:
- `build`: Build an image from a Containerfile or Dockerfile
- `cp`: Copy files/folders between a container and the local filesystem
- `exec`: Run a command in a running container
- `events`: Show Podman events
- `generate`: Generate structured data such as Kubernetes YAML or systemd units
- `images`: List local cached images
- `inspect`: Return low-level information on containers or images
- `kill`: Kill one or more running containers
- `load`: Load an image from a container TAR archive or stdin
- `login`: Log in to a container registry
- `logs`: Fetch the logs of a container
- `pod`: Manage pods
- `ps`: List running containers
- `pull`: Pull an image or a repository from a registry
- `push`: Push an image or a repository to a registry
- `restart`: Restart one or more containers
- `rm`: Remove one or more containers
- `rmi`: Remove one or more images
- `run`: Run a command in a new container
- `save`: Save one or more images to a TAR archive (streamed to stdout by default)
- `start`: Start one or more stopped containers
- `stop`: Stop one or more running containers
- `system`: Manage Podman (disk usage, container migration, REST API services, storage management, and pruning)
- `tag`: Create a `TARGET_IMAGE` tag that refers to `SOURCE_IMAGE`
- `unshare`: Run a command in a modified user namespace
- `volume`: Manage container volumes (list, pruning, creation, inspection)

- Podman CLI commands are compatible with Docker ones to help a smooth transition between the two tools.
- Users can choose to run a Podman service and make it listen to a Unix socket to expose native REST APIs.

Use Podman to create a socket endpoint on a path of preference and listen to API calls:
``` 
$ podman system service -–time 0 unix://tmp/podman.sock
```

- Default socket endpoint is `unix://run/podman/podman.sock` for rootful services and `unix://run/user/<UID>/podman/podman.sock` for non-root users.

Query Podman for the available local images:
``` 
$ curl --unix-socket /tmp/podman.sock \ http://d/v3.0.0/libpod/images/json | jq . 
```

Podman OpenAPI-compliant documentation: https://docs.podman.io/en/latest/\_static/api.html.

## Podman building blocks 

- Podman aims to adhere to open standards as much as possible
- Most of the runtime, build, storage, and networking components rely on community projects and standards. 

- The container life cycle is managed with the **libpod** library, included in the Podman main repository: https://github.com/containers/podman/tree/main/libpod
- The container runtime is based on the OCI specs implemented by OCI-compliant runtimes, such as **crun** and **runc**. 
- Image management implements the **containers/image** library (https://github.com/containers/container-libs/tree/main/image). 
	- Set of Go libraries used both by container engines and container registries.
- Container and image storage is implemented adopting the **containers/storage** library (https://github.com/containers/container-libs/tree/main/storage)
	- Go library to manage filesystem layers, container images, and container volumes at runtime.
- Image builds are implemented with Buildah (https://github.com/containers/buildah)
	- Both a binary tool and a library to build OCI images.
- Container runtime monitoring and communication with the engine is implemented with **Conmon**
	- A tool that monitors OCI runtimes, and is used by both Podman and **CRI-O** (https://github.com/containers/conmon).
- Container networking support is implemented through a Rust-based container network stack called **Netavark**. 
	- Introduced in Podman 4.0 along with **Container Network Interface** **(CNI)** as the new default. 
	- In Podman 5.0, CNI was declared deprecated. 
	- By default, Podman uses the basic `bridge` driver for the default network. 
	- Currently, bridge, **macvlan**, and **ipvlan** drivers are supported. 
	- The plugin-oriented approach allows users to implement their own Netavark plugins using the Netavark plugin API: https://github.com/containers/netavark/blob/main/plugin-API.md.

## The libpod library 

- Podman core foundations are based on the `libpod` library. 
- Contains all the necessary logic to orchestrate the container life cycle.
- Written in Go and accessed as a **Go package**.
- Intended to implement all the high-level functionalities of the engine. 
- According to the `libpod` and Podman documentation, its scope includes the following:
	- Managing container image format, which includes both OCI and Docker images. 
		- Includes the full image life cycle management
			- Authenticating and pulling from a container registry
			- Local storage of the image layers and metadata
			- Building of new images and pushing to remote registries.
	- Container life cycle management
		- Container creation (with all the necessary preliminary steps involved)
		- Running the container
		- All the other runtime functionalities
			- stop, kill, resume, and delete, process execution on running containers, and logging.
	- Managing both simple containers and pods, which are groups of sandboxed containers that share namespaces together (notably UTC, IPC, network, and optionally, PID as a recent feature) and are also managed together as a whole.
	- Supporting rootless containers and pods that can be executed by standard users with no need for privilege escalation.
	- Managing container resource isolation. 
		- Achieved at a low level with cgroups
		- Podman users can interact using CLI options during container execution to manage memory and CPU reservation or limit read/write rate on a storage device.
	- Supporting a CLI interface that can be used as a Docker-compatible alternative. 
		- Most Podman commands are the same as the Docker CLI.
	- Providing a Docker-compatible REST API with local Unix sockets (not enabled by default). 
		- Libpod REST APIs provide all the functionalities provided by the Podman CLI.

Libpod package 
- Interacts, at a lower level, with container runtimes, Conmon, and packages such as container/storage, container/image, Buildah, and Netavark.

## The runc and crun OCI container runtimes 

- A container engine takes care of the high-level orchestration of the container life cycle
- The low-level actions necessary to create and run the container are delivered by a container runtime.

**OCI Runtime Specification**: https://github.com/opencontainers/runtime-spec
- Industry standard
- The *Runtime and Lifecycle* document provides a full description of how the container runtime should handle the container creation and execution: https://github.com/opencontainers/runtime-spec/blob/master/runtime.md.

**runc** (https://github.com/opencontainers/runc) 
- Currently the most widely adopted OCI container runtime.
- Fully supports Linux containers and OCI runtime specs. 
- The project repository includes the **libcontainer** package, which is a Go package for creating containers with namespaces, cgroups, capabilities, and filesystem access controls. 

`libcontainer` package 
- Defines the inner logic and the low-level system interaction to bootstrap a container from scratch, from the initial isolation of namespaces to the execution as PID 1 of the binary program inside the container itself.

The runtime recalls the `libcontainer` library to fulfill the following tasks:
- Consume the container mount point and the container metadata provided by Podman.
- Interact with the kernel to start the container and execute the isolated process using the `clone()` and `unshare()` syscalls.
- Set up cgroup resource reservations.
- Set up SELinux Policy, Seccomp, and App Armor rules.

libcontainer
- Handles
	- Running processes.
	- Initialization of namespaces and file descriptors
	- Creation of the container `rootFS` and bind mounts
	- Exporting logs from container processes
	- Managing security restrictions with seccomp, SELinux, and AppArmor
	- Creating and mapping users and groups.
- Methods for the Linux OS that implement the interface are defined at https://github.com/opencontainers/runc/blob/master/libcontainer/container_linux.go.

**nsenter** package
- Handles Low-level execution of `clone()` and `unshare() syscall` to isolate the process namespaces
- By using the `nsexec()` function. https://github.com/opencontainers/runc/blob/master/libcontainer/nsenter/nsexec.c
	- C function embedded in the Go code, using **cgo**.

- Along with `runc`, many other container runtimes have been created. 

**crun** (https://github.com/containers/crun)
- A fast and low-memory-footprint OCI container runtime fully written in C. 
- The idea behind `crun` was to provide an improved OCI runtime that could leverage the C design approach for a cleaner and more lightweight runtime. 
- `runc` and `crun` can be used interchangeably by a container engine.
- Advantages against `runc`, such as the following:
	- **Smaller binary**: A `crun` build is approximately 50 times smaller than a `runc` build.
	- **Faster execution**: `crun` is faster on instrumenting the container than `runc` under the same execution conditions.
	- **Less memory usage**: `crun` consumes less than half the memory of `runc`. A smaller memory footprint is extremely helpful when dealing with massive container deployments or IoT appliances. It also allows you to set very low resource limits on containers (4 MB and possibly smaller).

- Can also be used as a library and integrated into other OCI-compliant projects. 
- Both `crun` and `runc` provide a command-line interface but are not meant to be used manually by end users, who are supposed to use a container engine such as Podman or Docker to manage the container life cycle.

To switch between the two runtimes in Podman, run a container using the `–runtime` flag to provide an OCI runtime binary path. 

Runs the container using `runc`:
``` $ podman --runtime /usr/bin/runc run --rm fedora echo "Hello World" ```

Runs the same container with the `crun` binary:
``` $ podman --runtime /usr/bin/crun run --rm fedora echo "Hello World" ```

- Runtime must be installed on the system for the --runtime flag to work.
- Both `crun` and `runc` support **eBPF** and **CRIU**.

**eBPF** (**Extended Berkeley Packet Filter**)
- Kernel-based technology
- Allows the execution of user-defined programs in the Linux kernel to add extra capabilities to the system without the need to recompile the kernel or load extra modules. 
- All eBPF programs are executed inside a sandbox virtual machine, and their execution is secure by design. Today, eBPF is gaining momentum and attracting industry interest, leading to wide adoption in different use cases, most notably, networking, security, observability, and tracing.

**Checkpoint Restore in Userspace** (**CRIU**) is a piece of software that enables users to freeze a running container and save its state to disk for further resumption. Data structures saved in memory are dumped and restored accordingly.

Another important architectural component used by Podman is Conmon, a tool for monitoring container runtime status. Let\'s investigate this in more detail in the next subsection.

## Conmon 

We may still have some questions about runtime execution.

How do Podman (the container engine) and `runc`/`crun` (the OCI container runtime) interact with each other? Which is responsible for launching the container runtime process? Is there a way to monitor the container execution?

Let\'s introduce the Conmon project (https://github.com/containers/conmon). Conmon is a monitoring and communication tool that sits between the container engine and the runtime. Every time a new container is created, a new instance of Conmon is launched. It detaches from the container manager process and runs daemonized, launching the container runtime as a child process.

If we attach a tracing tool to a Podman container, we can see in the following the order it\'s written in:

1. The container engine runs the Conmon process, which detaches and daemonizes itself. 2. The Conmon process runs a container runtime instance that starts the container and exits. 3. The Conmon process continues to run to provide a monitoring interface, while the manager/engine process has exited or detached.

The following diagram shows the logical workflow, from Podman execution to the running container:

![](images/B31467_2_5.png)

On a system with many running containers, users will find many instances of the Conmon process, one for every container created. In other words, Conmon acts as a small, dedicated daemon to the container.

Let\'s look at the following example, where a simple shell loop is used to create three identical nginx containers:

```
# for i in {1..3}; do podman run -d --rm docker.io/library/nginx; done 592f705cc31b1e47df18f71ddf922ea7e6c9e49217f00d1af8cf18c8e5557bde 4b1e44f512c86be71ad6153ef1cdcadcdfa8bcfa8574f606a0832c647739a0a2 4ef467b7d175016d3fa024d8b03ba44b761b9a75ed66b2050de3fec28232a8a7
# ps aux | grep conmon root 4272 0.0 0.5 9856 2456 ? Ss 20:22 0:00 /usr/bin/conmon --api-version 1 -c 775cb306658941f414298be6d185274dd8fc9e39e1a7bbe950b0ed87d40f749f -u 775cb306658941f414298be6d185274dd8fc9e39e1a7bbe950b0ed87d40f749f -r /usr/bin/crun [..omitted output] root 4341 0.0 0.5 9856 2464 ? Ss 20:22 0:00 /usr/bin/conmon --api-version 1 -c 5abde0da3e6f5c6a6f7ca4d7c4698124c8bc9c297efc835d7806e9a16281abd6 -u 5abde0da3e6f5c6a6f7ca4d7c4698124c8bc9c297efc835d7806e9a16281abd6 -r /usr/bin/crun [..omitted output] root 4409 0.0 0.5 9856 2460 ? Ss 20:22 0:00 /usr/bin/conmon --api-version 1 -c 9cd5e4461c9ab4d22baeabe0bfff3ab239d4d09b99ccd0eddf4d3c2cd2d16f48 -u 9cd5e4461c9ab4d22baeabe0bfff3ab239d4d09b99ccd0eddf4d3c2cd2d16f48 -r /usr/bin/crun [..omitted output] ```
```

After running the containers, a simple regular expression pattern applied to the output of the `ps aux` command shows three Conmon process instances.

Even if the initial Podman process is not running anymore (since there is no daemon), it is still possible to connect to the Conmon process and attach to the container. At the same time, Conmon exposes a way to attach to the container and container logs to log files or the systemd journal.

Conmon is a lightweight project written in C. It also provides Go language bindings to pass config structures between the manager and the runtime.

## Rootless containers 

One of the most interesting features of Podman is the capability to run rootless containers, which means that users without elevated privileges can run their own containers.

Rootless containers provide better security isolation and let different users run their own container instances independently. And, thanks to **fork/exec**, a daemonless approach adopted by Podman, rootless containers are amazingly easy to manage. A rootless container is simply run by the standard user with the usual commands and arguments, as in the following example:

``` $ podman run –d –-rm docker.io/library/nginx ```

When this command is issued, Podman creates a new user namespace and maps UIDs between the two namespaces using a **uid_map** file (see `man user_namespaces`). This method allows you to have, for example, a root user inside the container mapped to an ordinary user in the host.

Rootless containers and image data are stored under the user\'s home directory, usually under `$HOME/.local/share/containers/storage`.

Podman manages network connectivity for rootless containers in a different way than rootful containers. An in-depth technical comparison between rootless and rootful containers, especially from the network and security point of view, will be covered later in this book, in *[Chapter 10](#Chapter_10.xhtml#h1_248){.chapref}*.

After an in-depth analysis of the runtime workflow, it is useful to provide an overview of the OCI image specs used by Podman.

## OCI images 

Podman and the container/image package implement the OCI Image Format Specification. The full specification is available on GitHub at the following link and pairs with the OCI runtime specification: https://github.com/opencontainers/image-spec.

An OCI image is made of the following elements:

- Manifest
- An image index (optional)
- An image layout
- One or more filesystem layer changeset archive that will be unpacked to create a final filesystem
- An image configuration document to define layer ordering, as well as application arguments and environments
- A conversion document describing how the translation should occur
- An Artifacts Guidance document describing the spec usage for packaging content other than OCI images
- A reference named Descriptor to describe the type, metadata, and content address of referenced content

Let\'s see in detail what kinds of information and data are managed by the most relevant of the preceding elements.

### Manifest 

An image manifest specification should provide content-addressable images. The image manifest contains image layers and configurations for a specific architecture and operating system, such as Linux x86_64.

Here\'s the specification:

https://github.com/opencontainers/image-spec/blob/main/manifest.md

### Image index 

An image index is an object that contains a list of manifests related to different architectures (for example, amd64, arm64, or 386) and operating systems, along with custom annotations.

Here\'s the specification:

https://github.com/opencontainers/image-spec/blob/main/image-index.md

### Image layout 

The OCI image layout represents the directory structure of image blobs. The image layout also provides the necessary manifest location references, as well as the image index (in JSON format) and the image configuration. The image `index.json` contains the reference to the image manifest, stored as a blob in the OCI image bundle.

Here\'s the specification: https://github.com/opencontainers/image-spec/blob/main/image-layout.md

### Filesystem layers 

Inside an image, one or more layers are applied on top of each other to create a filesystem that the container can use.

At a low level, layers are packaged as TAR archives (with compression options with `gzip` and `zstd`). The filesystem layer implements the logic and order of layer stacking and how the changeset layers (layers containing file changes) are applied.

As described in the previous chapter, a copy-on-write or union filesystem has become a standard to manage stacking in a graph-like approach. To manage layer stacking, Podman uses **overlayfs** by default as a graph driver.

Here\'s the specification:

https://github.com/opencontainers/image-spec/blob/main/layer.md

### Image configuration 

An image configuration defines the image layer composition and the corresponding execution parameters, such as entry points, volumes, execution arguments, or environment variables, as well as additional image metadata.

The image JSON holding the configurations is an **immutable** object; changing it means creating a new derived image.

Here\'s the specification:

https://github.com/opencontainers/image-spec/blob/main/config.md

The following diagram represents an OCI image implementation, composed of image layer(s), image index, and image configuration:

![](images/B31467_2_6.png)

Let\'s inspect a realistic example from a basic, lightweight **alpine** image:

```
# tree alpine/ alpine/ ├── blobs │ └── sha256 │ ├── 03014f0323753134bf6399ffbe26dcd75e89c6a7429adfab392d64706649f07b │ ├── 696d33ca1510966c426bdcc0daf05f75990d68c4eb820f615edccf7b971935e7 │ └── a0d0a0d46f8b52473982a3c466318f479767577551a53ffc9074c9fa7035982e ├── index.json └── oci-layout ```
```

The directory layout contains an `index.json` file, with the following content:

```
 {
 "schemaVersion": 2, "manifests": [ { "mediaType": "application/vnd.oci.image.manifest.v1+json", "digest": "sha256:03014f0323753134bf6399ffbe26dcd75e89c6a7429adfab392d64706649f07b", "size": 348, "annotations": { "org.opencontainers.image.ref.name": "latest" } } ] }
```

The index contains a `manifests` array with only one item inside. The object `digest` is a SHA256 and corresponds to the filename as one of the blobs listed previously. The file is the image manifest and can be inspected:

```
# cat alpine/blobs/sha256/03014f0323753134bf6399ffbe26dcd75e89c6a7429adfab392d64706649f07b | jq {
 "schemaVersion": 2, "config": { "mediaType": "application/vnd.oci.image.config.v1+json", "digest": "sha256:696d33ca1510966c426bdcc0daf05f75990d68c4eb820f615edccf7b971935e7", "size": 585 }, "layers": [ { "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip", "digest": "sha256:a0d0a0d46f8b52473982a3c466318f479767577551a53ffc9074c9fa7035982e", "size": 2814446 } ] }
```

The manifest contains references to the image configuration and layers. In this particular case, the image has only one layer. Again, their digests correspond to the blob filenames listed before.

The config file shows image metadata, environment variables, and command execution. At the same time, it contains `DiffID` references to the layers used by the image and image creation information:

```
# cat alpine/blobs/sha256/696d33ca1510966c426bdcc0daf05f75990d68c4eb820f615edccf7b971935e7 | jq {
 "created": "2021-08-27T17:19:45.758611523Z", "architecture": "amd64", "os": "linux", "config": { "Env": [ "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" ], "Cmd": [ "/bin/sh" ] }, "rootfs": { "type": "layers", "diff_ids": [ "sha256:e2eb06d8af8218cfec8210147357a68b7e13f7c485b991c288c2d01dc228bb68" ] }, "history": [ { "created": "2021-08-27T17:19:45.553092363Z", "created_by": "/bin/sh -c #(nop) ADD file:aad4290d27580cc1a094ffaf98c3ca2fc5d699fe695dfb8e6e9fac20f1129450 in / " }, { "created": "2021-08-27T17:19:45.758611523Z", "created_by": "/bin/sh -c #(nop) CMD [\"/bin/sh\"]", "empty_layer": true } ] }
```

The image layer is the third blob file. This is a TAR archive that could be exploded and inspected. For space reasons, in this book, the example is limited to an inspection of the file type:

```
# file alpine/blobs/sha256/a0d0a0d46f8b52473982a3c466318f479767577551a53ffc9074c9fa7035982e alpine/blobs/sha256/a0d0a0d46f8b52473982a3c466318f479767577551a53ffc9074c9fa7035982e: gzip compressed data, original size modulo 2^32 5865472 ```
```

The result demonstrates that the file is a TAR gzipped archive.

# The main differences between Docker and Podman 

In the previous sections, we went through the key features of Docker and Podman, looking into the underlying layer, discovering the companion open source projects that made these two tools unique in their container engine role, but now it\'s time to compare them.

As we saw earlier, the significant difference between the two is that Docker has a daemon-centric approach while Podman instead has a daemonless architecture. The Podman binary acts as a command-line interface, as well as a container engine, and uses Conmon to orchestrate and monitor the container runtime.

Looking under the hood into the internals of both projects, we will also find many other differences, but, in the end, once the container has started, they both leverage OCI standard container runtimes, though with some differences: Docker uses `runc` while Podman uses `crun` in most distributions, with some exceptions; for example, Podman\'s default in Fedora or Red Hat Enterprise Linux 9 is `crun` while it still uses `runc` in the most conservative Red Hat Enterprise Linux 8, with `crun` as an option. Red Hat Enterprise Linux 10 only ships `crun` as the only runtime since it is considered fully stable and production-ready.

Despite the `crun` performance advantages described in the previous section, it is not the objective of this book to make a detailed performance comparison between the two. Readers interested in the topic will easily find literature about the performance differences between the two runtimes.

Another big gap that was recently filled by the Docker team was the rootless container. Podman was the first container engine to bring out this excellent feature that increases security and improves the usage of containers in many contexts, but, as we mentioned, this feature is now available in Docker too.

But let\'s go more practical in the next sections, by comparing them side by side through the command line first and then by running a container.

## Command-line interface comparison 

In this section, we will go through a side-by-side comparison, looking at the Docker and Podman CLIs.

Looking at the available commands for both CLIs, it is easy to spot the many similarities. The following table was truncated to improve readability:

 **Docker** **Podman** ------------------------ ----------------------------
 `attach` `attach` `build` `auto-update` `commit` `build` `cp` `commit` `create` `container` `diff` `cp` `events` `create` `exec` `diff` `export` `events` `history` `exec` `images` `export` `import` `generate` `info` `healthcheck` `inspect` `help` `kill` `history` `load` `image` `login` `images` `logout` `import` `logs` `info` `pause` `init` `port` `inspect` `…` `…`

Table 2.1 -- Comparison of Docker and Podman commands

As mentioned previously, Docker was born in 2013, while Podman only arrived 4 years later in 2017. Podman was built keeping in mind how experienced container administrators were with the most famous container engine available at that time: Docker. For this reason, the Podman development team decided not to change the *look and feel* of the command-line tools too much, to improve Docker users\' migration to the newly born Podman.

There was a claim, in fact, at the beginning of the distribution of Podman that if you have any existing scripts that run Docker, you can create an alias and it should work (`alias docker=podman`). There was also a package that places a *fake* Docker command under `/usr/bin` that points to the *Podman* binary instead. For this reason, if you are a Docker user, you can expect a smooth transition to Podman once you are ready.

Another important point is that the images created with Docker are compatible with the OCI standard, so you can easily migrate or pull any image you previously used again with Docker.

If we take a deep look into the command options available for Podman, you will notice that there are some additional commands that are not present in Docker, while some others are missing.

For example, Podman can manage, along with containers, pods (the name Podman is quite promising here). The pod concept was introduced with Kubernetes and represents the smallest execution unit in a Kubernetes cluster.

With Podman, users can create empty pods and then run containers inside them easily using the following command:

``` $ podman pod create --name mypod $ podman run –-pod mypod –d docker.io/library/nginx ```

This is not as easy with Docker, where users must first run a container and then create new ones attaching to the network namespace of the first container.

Podman has additional features that could help users to move their containers in Kubernetes environments. Using the `podman generate kube` command, Podman can create a Kubernetes YAML file for a running container that can be used to create a pod inside a Kubernetes cluster.

Running containers as systemd services is equally easy by placing the `podlet` command in front of the original `podman run` command. This takes a running container or pod and generates a systemd unit file that can be used to automatically start the service at system startup.

Here\'s a notable example: the **OpenStack** project, an open source cloud computing infrastructure, adopted Podman as the default manager for its containerized services when deployed with TripleO. All the services are executed by Podman and orchestrated by systemd in the control plane and compute nodes.

Having checked the surface of these container engines and having looked at their command lines, let\'s recap the under-the-hood differences in the next section.

## Running a container 

Running a container in a Docker environment, as we mentioned earlier, consists of using the Docker command-line client to communicate with the Docker daemon that will do the actions required to get the container up and running. Just to summarize the concepts we explained in this chapter, we can take a look at the following diagram:

![](images/B31467_2_7.png)

Podman, instead, interacts directly with the image registry, storage, and with the Linux kernel through the container runtime process (not a daemon), with Conmon as a monitoring process executed between Podman and the OCI runtime, as we can schematize in the following diagram:

![](images/B31467_2_8.png)

The core difference between the two architectures is the daemon-centric Docker vision versus the fork/exec approach of Podman.

This book does not get into the pros and cons of the Docker daemon architecture and features. However, it is fair to say that a significant number of Docker users were concerned about this daemon-centric approach for many reasons. Here are some examples:

- The daemon could be a single point of failure
- Resource consumption
- Poor support for rootless containers due to the need for per-user Docker and containerd daemons

Despite the architectural differences and the aliasing solutions described before to easily migrate projects without changing any script, running a container from the command line with Docker or Podman is pretty much the same thing for the end user:

``` $ docker run –d -–rm docker.io/library/nginx $ podman run –d -–rm docker.io/library/nginx ```

For the same reason, most of the command-line arguments of CLI commands have been kept as close as possible to the original version in Docker.

# Further reading 

For more information about the topics covered in this chapter, you can refer to the following:

- Rootless containers with Podman: The basics: https://developers.redhat.com/blog/2020/09/25/rootless-containers-with-podman-the-basics
- How to transition from Docker to Podman: https://developers.redhat.com/blog/2020/11/19/transitioning-from-docker-to-podman
- cgroup v2: https://github.com/opencontainers/runc/blob/master/docs/cgroup-v2.md
- An introduction to `crun`, a fast and low-memory footprint container runtime: https://www.redhat.com/sysadmin/introduction-crun
- eBPF Documentation: https://ebpf.io/what-is-ebpf/
