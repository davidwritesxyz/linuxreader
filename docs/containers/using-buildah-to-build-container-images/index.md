# 

# Introducing Buildah, Podman's companion 

Podman does an excellent job in plain builds with Dockerfiles/Containerfiles and helps teams to preserve their previously implemented build pipelines without the need for new investments.

However, when it comes to more specialized build tasks or when users need more control over the build workflow, with the option of including scripting logic, the Dockerfile/Containerfile approach shows its limitations. Communities struggled to find alternative building approaches that could overcome the rigid, workflow-based logic of Dockerfiles/Containerfiles.

The same community that develops Podman brought to life the Buildah (pronounced *build-ah*) project, a tool to manage OCI builds with support for multiple building strategies. Images created with Buildah are fully portable and compatible with Docker, and all engines are compliant with the OCI image and runtime specs.

Buildah is an open source project released under the Apache 2.0 license. Sources are available on GitHub at the following URL: https://github.com/containers/buildah.

Buildah is complementary to Podman, which borrows its build logic by vendoring its libraries to implement basic build functionalities against Dockerfiles and Containerfiles. The final Podman binary, which is compiled in Go as a statically linked single file, embeds Buildah packages to manage the build steps.

Buildah uses the `containers/image` project (https://github.com/containers/image) to manage an image\'s life cycle and its interaction with registries, and the `containers/storage` project (https://github.com/containers/storage) to manage images and container filesystem layers. These libraries are the same that Podman uses, and they have both now been moved into the `containers/containers-libs` project (https://github.com/containers/container-libs).

The advanced build strategy of Buildah is based on the parallel support for traditional Dockerfile/Containerfile-based builds, and for builds driven by native Buildah commands that replicate the Dockerfile instructions.

By replicating Dockerfile instructions in standard commands, Buildah becomes a scriptable tool that can be interpolated with custom logic and native shell constructs such as conditionals, loops, or environment variables. For example, the `RUN` instruction in a Dockerfile can be replaced with a `buildah run` command.

If teams need to preserve the build logic implemented in previous Dockerfiles, Buildah offers the `buildah build` (or its alias, `buildah bud`) command, which builds the image reading from the provided Dockerfile/Containerfile, as in a typical `podman build` command.

Buildah can smoothly run in rootless mode to build images; this is a valuable, highly demanded feature from a security point of view. No Unix sockets are necessary to run a build. At the beginning of this chapter, we explained how builds are always based on containers; Buildah is not exempt from this behavior, and all its builds are executed inside working containers, starting on top of the base image.

The following list provides a non-exhaustive description of the most frequently used commands in Buildah:

- `buildah from`: Initializes a new working container on top of a base image. It accepts the `buildah from [options] <image>` syntax. An example of this command is `$ buildah from fedora`.
- `buildah run`: This is equivalent to the `RUN` instruction of a Dockerfile; it runs a command inside a working container. This command accepts the `buildah run [options] [--] <container> <command>` syntax. The `--` (double dash) option is necessary to separate potential options from the effective container command. An example of this command is `buildah run <containerID> -- dnf install -y nginx`.
- `buildah config`: This command configures image metadata. It accepts the `buildah config [options] <container>` format. The options available for this command are associated with the various Dockerfile instructions that do not modify filesystem layers but set some container metadata -- for instance, the setup of the `entrypoint` container. An example of this command is `buildah config --entrypoint/entrypoint.sh <containerID>`.
- `buildah add`: This is equivalent to the `ADD` instruction of the Dockerfile; it adds files, directories, and even URLs to the container. It supports the `buildah add [options] <container> <src> [[src …] <dst>` syntax and allows you to copy multiple files in one single command. An example of this command is `buildah add <containerID> index.php /var/www.html`.
- `buildah copy`: This is the same as the Dockerfile `COPY` instruction; it adds files, URLs, and directories to the container. It supports the `buildah copy [options] <container> <src> [[src …] <dst>` syntax. An example of this command is `buildah copy <containerID> entrypoint.sh /`.
- `buildah commit`: This commits a final image out of a working container. This command is usually the last executed one. It supports the `buildah copy [options] <container> <image_name>` syntax. The container image created from this command can be later tagged and pushed to a registry. An example of this command is `buildah commit <containerID> <myhttpd>`.
- `buildah build`: The equivalent command of the classic `podman build` command. This command takes Dockerfiles or Containerfiles as arguments, along with the build directory path. It accepts the `buildah build [options] [context]` syntax and the `buildah bud` command alias. An example of this command is `buildah build -t <imageName> .`.
- `buildah containers`: This lists the active working container involved in Buildah builds, along with the base image used as a starting point. Equivalent commands are `buildah ls` and `buildah ps`. The supported syntax is `buildah containers [options]`. An example of this command is `buildah containers`.
- `buildah rm`: This is used to remove working containers. The `buildah delete` command is equivalent. The supported syntax is `buildah rm <container>`. This command has only one option, the `-all, -a` option, to remove all the working containers. An example of this command is `buildah rm <containerID>`.
- `buildah mount`: This command can be used to mount a working container root filesystem. The accepted syntax is `buildah mount [containerID … ]`. When no argument is passed, the command only shows the currently mounted containers. A practical example of this command used to mount a container filesystem is `buildah mount <containerID>`. This can be used to avoid `buildah add` and `buildah copy` and instead directly change the container filesystem yourself.
- `buildah images`: This lists all the available images in the local host cache. The accepted syntax is `buildah images [options] [image]`. Custom output formats such as JSON are available. An example of this command is `buildah images --json`.
- `buildah tag`: This applies a custom name and tags to an image in the local store. The syntax follows the `buildah tag <name> <new-name>` format. An example of this command is `buildah tag myapp quay.io/packt/myapp:latest`.
- `buildah push`: This pushes a local image to a remote private or public register, or local directories in Docker or OCI format. The command syntax is `buildah push [options] <image> [destination]`. Examples of this command include `buildah push quay.io/packt/myapp:latest`,` buildah push <imageID> docker://<URL>/repository:tag`, and `buildah push <imageID> oci:</path/to/dir>:image:tag`.
- `buildah pull`: This pulls an image from a registry, an OCI archive, or a directory. The syntax includes `buildah pull [options] <image>`. Examples of this command include `buildah pull <imageName>`, `buildah pull docker://<URL>/repository:tag`, and `buildah pull dir:</path/to/dir>`.

All the commands described previously have their corresponding `man` page, with the `man buildah-<command>` pattern. For example, to read documentation details about the `buildah run` command, just type `man buildah-run` on the terminal.

The next example shows basic Buildah capabilities. A Fedora base image is customized to run an `httpd` process:

``` $ container=$(buildah from fedora) $ buildah run $container -- dnf install -y httpd; dnf clean all $ buildah config --cmd "httpd -DFOREGROUND" $container $ buildah config --port 80 $container $ buildah commit $container myhttpd $ buildah tag myhttpd registry.example.com/myhttpd:v0.0.1 ```

The preceding commands will produce an OCI-compliant, portable image with the same features as an image built from a Dockerfile, all in a few lines that can be included in a simple script.

 note **Good to** **know**

While the preceding example uses `buildah run` to execute commands inside the container, the real *secret weapon* of Buildah is the `buildah mount` command.

One of the primary reasons developers choose Buildah over standard Dockerfiles is the ability to interact with the container\'s filesystem directly from the host. When you run `buildah mount $container`, the container\'s root filesystem is mapped to a path on your host machine. This unlocks several powerful workflows.

We will now focus on the first command:

``` $ container=$(buildah from fedora) ```

The `buildah from` command pulls a Fedora image from one of the allowed registries and spins up a working container from it, returning the container name. Instead of simply having it printed on standard output, we will capture the name with shell expansion syntax. From now on, we can pass the `$container` variable, which holds the name of the generated container, to the subsequent commands. Therefore, the build commands will be executed inside this working container. This is quite a common pattern and is especially useful to automate Buildah commands in scripts.

There is a subtle difference between the concept of containers in Buildah and Podman. Both adopt the same technology to create containers, but Buildah containers are short-lived entities that are created to be modified and committed, while Podman containers are supposed to run long-living workloads.

Also, please consider that commands such as `podman ps` do not show Buildah containers by default and require the `--external` option.

The flexible and embeddable nature of this approach is remarkable --
Buildah commands can be included anywhere, and users can choose between a fully automated build process and a more interactive one.

For example, Buildah can be easily integrated with **Ansible**, the open source automation engine, to provide automated builds using native connection plugins that enable communication with working containers.

You can choose to include Buildah inside a CI pipeline (such as **Jenkins**, **Tekton**, or **GitLab CI/CD**) to gain full control of the build and integration tasks. Tekton, the cloud-native CI/CD tool (https://tekton.dev/), offers a collection of template tasks, the building block of Tekton pipelines, on the Tekton Hub repository (https://hub.tekton.dev/); Buildah tasks are also available on Tekton Hub (https://hub.tekton.dev/tekton/task/buildah). In addition, Buildah can be run inside a container without issue, allowing it to run in many other CI pipelines, even if they do not normally support container builds.

Buildah is also included in larger projects of the cloud-native community, such as the **Shipwright** project (https://github.com/shipwright-io/build).

Shipwright is an extensible build framework for Kubernetes that provides the flexibility of customizing image builds using custom resource definitions and different build tools. Buildah is one of the available solutions that you can choose when designing your build processes.

We will see more detailed and richer examples in the next subsections. Now that we have seen an overview of Buildah\'s capabilities and use cases, let\'s dive into the installation and environment preparation steps.

# Preparing our environment 

Buildah is available on different distributions and can be installed using the respective package managers. This section provides a non-exhaustive list of installation examples on the major distributions. For the sake of clarity, it is important to reiterate that the book lab environments were all based on Fedora 40:

- **Fedora**: To install Buildah on Fedora and every other Linux distribution that uses the DNF package manager, run the following `dnf` command:

 ``` {.programlisting .snippet-con-one} $ sudo dnf -y install buildah ```

- **Debian**: To install Buildah on Debian Bullseye or later, run the following `apt-get` commands:

 ``` {.programlisting .snippet-con-one} $ sudo apt-get update $ sudo apt-get -y install buildah ```

- **CentOS Stream 9**: To install Buildah on CentOS Stream 9, run the following `yum` command:

 ``` {.programlisting .snippet-con-one} $ sudo dnf install -y buildah ```

- **RHEL8**: To install Buildah on RHEL8, run the following `yum module` commands:

 ``` {.programlisting .snippet-con-one} $ sudo yum module enable -y container-tools:1.0 $ sudo yum module install -y buildah ```

- **RHEL9**: To install Buildah on RHEL9, install the `container-tools` meta-package:

 ``` {.programlisting .snippet-con-one} $ sudo dnf -y install container-tools ```

- **Arch Linux**: To install Buildah on Arch Linux, run the following `pacman` command:

 ``` {.programlisting .snippet-con-one} $ sudo pacman -S buildah ```

- **Ubuntu**: To install Buildah on Ubuntu 20.10 or later, run the following `apt-get` commands:

 ``` {.programlisting .snippet-con-one} $ sudo apt-get -y update $ sudo apt-get -y install buildah ```

- **Gentoo**: To install Buildah on Gentoo, run the following `emerge` command:

 ``` {.programlisting .snippet-con-one} $ sudo emerge app-emulation/libpod ```

- **Build from source**: Buildah can also be built from the source. For the purpose of this book, we will keep the focus on simple deployment methods, but if you\'re curious, you will find the following guide useful to try out your own builds: https://github.com/containers/buildah/blob/main/install.md#building-from-scratch.

Finally, Buildah can be deployed as a container, and builds can be executed inside it with a nested approach. This process will be covered in greater detail in *[Chapter 7](#Chapter_7.xhtml#h1_183){.chapref}*, *Integrating* *with Existing Application Build Processes*.

After installing Buildah on our host, we can move on to verifying our installation.

## Verifying the installation 

After installing Buildah, we can now run some basic test commands to verify the installation.

To see all the available images in the host local store, use the following commands:

``` $ buildah images
# buildah images 
```

The image list will be the same as the one printed by the `podman images` command, since they share the same local store.

Also note that the two commands are executed as an unprivileged user and as root, pointing respectively to the user rootless local store and the system-wide local store.

We can run a simple test build to verify the installation. This is a good chance to test a basic build script whose only purpose is to verify whether Buildah is able to fully run a complete build.

For the purpose of this book (and for fun), we have created the following simple test script that creates a minimal Python 3 image:

```
#!/bin/bash

BASE_IMAGE=alpine TARGET_IMAGE=python3-minimal

if [ $UID != 0 ]; then echo "### Running build test as unprivileged user" else echo "### Running build test as root" fi

echo "### Testing container creation" container=$(buildah from $BASE_IMAGE) if [ $? -ne 0 ]; then echo "Error initializing working container" fi

echo "### Testing run command" buildah run $container apk add --update python3 py3-pip if [ $? -ne 0 ]; then echo "Error on run build action" fi

echo "### Testing image commit" buildah commit $container $TARGET_IMAGE if [ $? -ne 0 ]; then echo "Error committing final image" fi

echo "### Removing working container" buildah rm $container if [ $? -ne 0 ]; then echo "Error removing working container" fi

echo "### Build test completed successfully!" exit 0 
```

The same test script can be executed by a non-privileged user and by root.

We can verify the newly built image by running a simple container that executes a Python shell:

``` $ podman run -it python3-minimal /usr/bin/python3 Python 3.12.7 (main, Oct 7 2024, 11:30:19)Python 3.9.5 (default, May 12 2021, 20:44:22) [GCC 13.2.1 2024030910.3.1 20210424] on linux Type "help", "copyright", "credits" or "license" for more information. >>> ```

After successfully testing our new Buildah installation, let\'s inspect the main configuration files used by Buildah.

## Buildah configuration files 

The main Buildah configuration files are the same ones used by Podman. They can be leveraged to customize the behavior of the working containers executed in builds.

On Fedora, these config files are installed by the `containers-common` package, and we already covered them in the *Prepare your environment* section in *[Chapter 3](#Chapter_3.xhtml#h1_83){.chapref}*, *Running* *the First Container*.

The main config files used by Buildah are as follows:

- `/usr/share/containers/mounts.conf`: This config file defines the files and directories that are automatically mounted inside a Buildah working container
- `/etc/containers/registries.conf`: This config file has the role of managing registries allowed to be accessed for image searches, pulls, and pushes
- `/usr/share/containers/policy.json`: This JSON config file defines image signature verification behavior
- `/usr/share/containers/seccomp.json`: This JSON config file defines the allowed and prohibited syscalls to a containerized process
- `/usr/share/containers/containers.conf`: This config file contains many Podman-specific settings, but it\'s also used by Buildah

In this section, we have learned how to prepare the host environment to run Buildah. In the next section, we are going to identify the possible build strategies that can be implemented with Buildah.

# Choosing our build strategy 

There are basically three types of build strategies that we can use with Buildah:

- Building a container image starting from an existing base image
- Building a container image starting from scratch
- Building a container image starting from a Dockerfile

We have already provided an example of the build strategy from an existing base image in the *Introducing* *Buildah, Podman\'s companion* section. Since this strategy is pretty similar from a workflow point of view to building from scratch, we will focus our practical examples on the last one, which provides great flexibility to create a small footprint and secure images.

Before going through the various technical details in the next section, let\'s start exploring all these strategies at a high level.

Even though we can find a lot of prebuilt container images available on the most popular public container registries, sometimes we might not be able to find a particular configuration, setup, or bundle of tools and services for our containers; that is why container image creation becomes a really important step that we need to practice. In addition to that, the image creation also allows bundling your own configuration files to customize an image for your exact use.

Also, security constraints often require us to implement images with reduced attack surfaces, and therefore, DevOps teams must know how to customize every step of the build process to achieve this result. In enterprise contexts, for optimization and security reasons, images are almost always created from scratch or using minimal base images ready to be customized.

With this awareness in mind, let\'s start with the first build strategy.

## Building a container image starting from an existing base image 

Let\'s imagine finding a well-done prebuilt container image for our favorite application server that our company is widely using. All the configurations for this container image are okay, and we can attach storage to the right mount points to persist the data and so on, but sooner or later, we may realize that some particular tools that we use for troubleshooting are missing in the container image, or that some libraries are missing that should be included!

In another scenario, we could be happy with the prebuilt image but still need to add custom content to it -- for example, the customer application.

What would be the solution in those cases?

In this first use case, we can extend the existing container image, adding stuff and editing the existing files to suit our purposes. In the previous basic examples, Fedora and Alpine images were customized to serve different purposes. Those images were generic OS filesystems with no specific purpose, but the same concept can be applied to a more complex image.

In the second use case, we can customize an image -- for example, the default library, `Httpd`. We can install PHP modules and then add our application\'s PHP files, producing a new image with our custom content already built in.

We will see in the next sections that we can extend an existing container image.

Let\'s move on to the second strategy.

## Building a container image starting from scratch 

The previous strategy would be enough for many common situations, where we can find a prebuilt image to start working with, but sometimes it may be that the particular use case, application, or service that we want to containerize is not so common or widely used.

Imagine having a custom legacy application that requires some old libraries and tools that are no longer included on the latest Linux distribution, or that may have been replaced by more recent ones. In this scenario, you might need to start from an empty container image and add all the necessary stuff for your legacy application piece by piece.

Finally, `FROM scratch` is also a way of distributing super-minimal container images that contain only the files necessary to run your container. You can even make single-file images with only a single static binary inside them.

We have learned in this chapter that, actually, we will always start from a sort of initial container image, so this strategy and the previous one are pretty much the same.

Let\'s move on to the third and final strategy.

## Building a container image starting from a Dockerfile 

In *[Chapter 1](#Chapter_1.xhtml#h1_14){.chapref}*, *Introduction to Container Technology*, we talked about container technology history and how Docker gained momentum in that context. Podman was born as an alternative evolution project of the great concepts that Docker helped to develop until now. One of the great innovations that Docker created in its own project history is, for sure, the Dockerfile.

Looking into this strategy at a high level, we can affirm that even when using a Dockerfile, we will arrive at one of the previous build strategies. The reality is not far away from the latest assumption we made, because Buildah, under the hood, will parse the Dockerfile, and it will build the container that we briefly introduced for previous build strategies.

So, in summary, are there any differences or advantages we need to consider when choosing our default build strategy? Obviously, there is no ultimate answer to this question. First of all, we should always look into the container communities, searching for some prebuilt image that could help our *build* process; on the other hand, we can always fall back on the *build-from-scratch* process. Last but not least, we can consider Dockerfile for easily distributing and sharing our build steps with our development group or the whole container community.

This ends our quick high-level introduction; we can now move on to the practical examples!

# Building images from scratch 

Before going into the details of this section and learning how to build a container image from scratch, let\'s make some tests to verify that the installed Buildah is working properly.

First of all, let\'s check whether our Buildah image cache is empty:

```
# buildah images REPOSITORY TAG IMAGE ID CREATED SIZE
# buildah containers -a CONTAINER ID BUILDER IMAGE ID IMAGE NAME CONTAINER NAME 
```

Podman and Buildah share the same container storage; for this reason, if you previously ran any other example shown in this chapter or book, you will find that your container storage cache is not that empty!

As we learned in the previous section, we can leverage the fact that Buildah will output the name of the just-created working container to easily store it in an environment variable and use it once needed. Let\'s create a brand-new container from scratch:

```
# buildah from scratch
# buildah images REPOSITORY TAG IMAGE ID CREATED SIZE
# buildah containers CONTAINER ID BUILDER IMAGE ID IMAGE NAME CONTAINER NAME af69b9547db9 * scratch working-container 
```

As you can see, we used the special `from scratch` keywords that tell Buildah to create an empty container with no data inside it. If we run the `buildah images` command, we will note that this special image is not listed.

Let\'s check whether the working container built from scratch is really empty:

```
# buildah run working-container bash 2021-10-26T20:15:49.000397390Z: executable file `bash` not found in $PATH: No such file or directory error running container: error from crun creating container for [bash]: : exit status 1 error while running runtime: exit status 1 
```

No executable was found in our empty container -- what a surprise! The reason is that the working container has been created on an empty filesystem.

Let\'s see how we can easily fill this empty container. In the following example, we will interact directly with the underlying storage, using the package manager of our host system to install the binaries and the libraries needed for running a `bash` shell in our container image.

First of all, let\'s instruct Buildah to mount the container storage and check where it resides:

```
# buildah mount working-container /var/lib/containers/storage/overlay/b5034cc80252b6f4af2155f9e0a2a7e65b77dadec7217bd2442084b1f4449c1a/merged 
```

 note **Good to** **know**

If you start the build in rootless mode, Buildah will run the mount in a different namespace, and for this reason, the mounted volume might not be accessible from the host when using a driver different from `vfs`. Alternatively, you can run the commands in a `podman unshare` shell.

Great! Now that we have found it, we can leverage the host package manager to install all the needed packages in this `root` folder, which will be the `root` path of our container image:

```
# scratchmount=$(buildah mount working-container)
# dnf install --installroot $scratchmount --releasever 40 bash coreutils --setopt install_weak_deps=false -y 
```

If you are running the previous command on a Fedora release different from version 40, (for example, version 39), then you need to import the GPG public keys of Fedora 40 or use the additional option --
`--nogpgcheck`.

First of all, we will save the very long directory path in an environment variable and then execute the `dnf` package manager, passing the just-obtained directory path as the install root directory, setting the release version of our Fedora OS, specifying the packages that we want to install (`bash` and `coreutils`), and finally, disabling weak dependencies, accepting all the changes to the system.

The command should end up with a `Complete!` statement; once done, let\'s try again with the same command that earlier in this section we saw was failing:

```
# buildah run working-container bash bash-5.2# cat /etc/fedora-release Fedora release 40 (Forty) 
```

It worked! We just installed a Bash shell in our empty container. Let\'s see now how to finish our image creation with some other configuration steps. First of all, to our final container image, we need to add a command to be run once it is up and running. For this reason, we will create a Bash script file with some basic commands:

```
# cat command.sh
#!/bin/bash cat /etc/fedora-release /usr/bin/date 
```

We have created a Bash script file that prints the container Fedora release and the system date. The file must have execute permissions before being copied:

```
# chmod +x command.sh 
```

Now that we have filled up our underlying container storage with all the needed base packages, we can unmount the `working-container` storage and use the `buildah copy` command to inject files from the host to the container:

```
# buildah unmount working-container af69b9547db93a7dc09b96a39bf5f7bc614a7ebd29435205d358e09ac99857bc
# buildah copy working-container ./command.sh /usr/bin 659a229354bdef3f9104208d5812c51a77b2377afa5ac819e3c3a1a2887eb9f7 
```

The `buildah copy` command gives us the ability to work with the underlying storage without worrying about mounting it or handling it under the hood.

We are now ready to complete our container image by adding some metadata to it:

```
# buildah config --cmd /usr/bin/command.sh working-container
# buildah config --created-by "Podman for DevOps example" working-container
# buildah config --label name=fedora-date working-container 
```

We started with the `cmd` option, and after that, we added some descriptive metadata. We can finally commit `working-container` into an image!

```
# buildah commit working-container fedora-date Getting image source signatures Copying blob 939ac17066d4 done Copying config e24a2fafde done Writing manifest to image destination Storing signatures e24a2fafdeb5658992dcea9903f0640631ac444271ed716d7f749eea7a651487 
```

Let\'s clean up the environment and check the available container images on the host:

```
# buildah rm working-container af69b9547db93a7dc09b96a39bf5f7bc614a7ebd29435205d358e09ac99857bc 
```

We can now inspect the details of the just-created container image:

```
# podman images REPOSITORY TAG IMAGE ID CREATED SIZE localhost/fedora-date latest e24a2fafdeb5 About a minute ago 366 MB
# podman inspect localhost/fedora-date:latest [...omitted output] "Labels": { "io.buildah.version": "1.37.3", "name": "fedora-date" }, "Annotations": { "org.opencontainers.image.base.digest": "", "org.opencontainers.image.base.name": "" }, "ManifestType": "application/vnd.oci.image.manifest.v1+json", "User": "", "History": [ { "created": "2024-11-01T13:51:50.170096562Z", "created_by": "Podman for DevOps example" } ], "NamesHistory": [ "localhost/fedora-date:latest" ] } ]
```

As we can see from the previous output, the container image has a lot of metadata that can tell us many details. Some of them we set through the previous commands, such as the `created_by`, `name`, and `Cmd` tags; the other tags are populated automatically by Buildah.

Finally, let\'s run our brand-new container image with Podman!

```
# podman run -ti localhost/fedora-date:latest Fedora release 40 (Forty) Tue Nov 01 21:18:29 UTC 2024 
```

This ends our journey in creating a container image from scratch. As we saw, this is not a typical method for creating a container image; in many scenarios and for various use cases, it can be enough to start with an OS base image, such as `from fedora` or `from alpine`, and then add the required packages, using the respective package managers available in those images.

 note **Good to** **know**

Some Linux distributions also provide base container images in a **minimal** flavor (for example, `fedora-minimal`) that reduces the number of packages installed, as well as the size of the target container image. For more information, refer to https://www.docker.com/ and https://quay.io/!

Let\'s now inspect how to build images from Dockerfiles with Buildah.

# Building images from a Dockerfile 

As we described earlier in this chapter, the Dockerfile can be an easy option to create and share the build steps for creating a container image, and for this reason, it is really easy to find a lot of source Dockerfiles on the net.

The first step of this activity is to build a simple Dockerfile to work with. Let\'s create a Dockerfile for creating a containerized web server:

``` {.programlisting .snippet-code}
# mkdir webserver
# cd webserver/ [webserver]# vi Dockerfile [webserver]# cat Dockerfile
# Start from latest fedora container base image FROM fedora:latest

# Update the container base image RUN echo "Updating all fedora packages"; dnf -y update; dnf -y clean all

# Install the httpd package RUN echo "Installing httpd"; dnf -y install httpd

# Expose the http port 80 EXPOSE 80

# Set the default command to run once the container will be started CMD ["/usr/sbin/httpd", "-DFOREGROUND"] 
```

Looking at the previous output, we first created a new directory, and inside, we created a text file named `Dockerfile`. After that, we inserted the various keywords and steps commonly used in the definition of a brand-new Dockerfile; every step and keyword has a dedicated description comment on top, so the file should be easy to read.

Just to recap, these are the steps contained in our brand-new Dockerfile:

1. Start from the latest Fedora container base image. 2. Update all the packages for the container base image. 3. Install the `httpd` package. 4. Expose HTTP port `80`. 5. Set the default command to run once the container is started.

As seen previously in this chapter, Buildah provides a dedicated `buildah build` command to start a build from a Dockerfile.

Let\'s see how it works:

``` 
[webserver]
# buildah build -f Dockerfile -t myhttpdservice . STEP 1/6: FROM fedora:latest Resolved "fedora" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf) Trying to pull registry.fedoraproject.org/fedora:latest... Getting image source signatures Copying blob 944c4b241113 done Copying config 191682d672 done Writing manifest to image destination Storing signatures STEP 2/6: MAINTAINER podman-book # this should be an email STEP 3/6: RUN echo "Updating all fedora packages"; dnf -y update; dnf -y clean all Updating all fedora packages Fedora 40
- x86_64 16 MB/s | 74 MB 00:04 ... STEP 4/6: RUN echo "Installing httpd"; dnf -y install httpd Installing httpd Fedora 40
- x86_64 20 MB/s | 74 MB 00:03 ... STEP 5/6: EXPOSE 80 STEP 6/6: CMD ["/usr/sbin/httpd", "-DFOREGROUND"] COMMIT myhttpdservice Getting image source signatures Copying blob 7500ce202ad6 skipped: already exists Copying blob 51b52d291273 done Copying config 14a2226710 done Writing manifest to image destination Storing signatures
--> 14a2226710e Successfully tagged localhost/myhttpdservice:latest 14a2226710e7e18d2e4b6478e09a9f55e60e0666dd8243322402ecf6fd1eaa0d 
```

As we can see from the previous output, we pass the following options to the `buildah build` command:

- `-f`: To define the name of the Dockerfile. The default filename is `Dockerfile`, so in our case, we can omit this option because we named the file as the default one.
- `-t`: To define the name and the tag of the image we are building. In our case, we are only defining the name. The image will be tagged as `latest` by default.

Finally, as the last option, we need to specify the directory where Buildah needs to work and search for the Dockerfile. In our case, we are passing the`.` Directory, which represents the current working directory.

Of course, these are not the only options that Buildah gives us to configure the build; we will see some of them later in this section.

Coming back to the command we just executed, as we can see from the output, all the steps defined in the Dockerfile have been executed in the exact order and printed with a given fractional number to show the intermediate steps against the total number. In total, six steps were executed.

We can check the result of our command by listing the images with the `buildah images` command:

``` [webserver]# buildah images REPOSITORY TAG IMAGE ID CREATED SIZE localhost/myhttpdservice latest 14a2226710e7 2 minutes ago 497 MB ```

As we can see, our container image has just been created with the `latest` tag; let\'s try to run it:

```
# podman run -d localhost/myhttpdservice:latest 133584ab526faaf7af958da590e14dd533256b60c10f08acba6c1209ca05a885
# podman logs 133584ab526faaf7af958da590e14dd533256b60c10f08acba6c1209ca05a885 AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.88.0.4. Set the 'ServerName' directive globally to suppress this message

# curl 10.88.0.4 <!doctype html> <html> <head> <meta charset='utf-8'> <meta name='viewport' content='width=device-width, initial-scale=1'> <title>Test Page for the HTTP Server on Fedora</title> <style type="text/css"> ... 
```

Looking at the output, we just ran our container in detached mode; after that, we inspected the logs to find out the IP address that we need to pass as an argument for the `curl` test command.

We just run the container as the root user on our workstation, and the container just received an internal IP address on Podman\'s container network interface. We can check that the IP address is part of that network by running the following commands:

```
# ip a show dev cni-podman0 14: cni-podman0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000 link/ether c6:bc:ba:7c:d3:0c brd ff:ff:ff:ff:ff:ff inet 10.88.0.1/16 brd 10.88.255.255 scope global cni-podman0 valid_lft forever preferred_lft forever inet6 fe80::c4bc:baff:fe7c:d30c/64 scope link valid_lft forever preferred_lft forever 
```

As we can see, the container\'s IP address was taken from the network reported in the previous `10.88.0.1/16` output.

As we anticipated, the `buildah build` command has a lot of other options that can be useful while developing and creating brand-new container images. Let\'s explore one of them that is worth mentioning: `--layers`.

We already learned how to use this option with Podman earlier in this chapter. Starting from version 1.2 of Buildah, the development team added this great option that gives us the ability to enable or disable the layers\' caching mechanism. The default configuration sets the `--layers` option to `false`, which means that Buildah will not keep intermediate layers, resulting in a build that squashes all the changes in a single layer.

It is also possible to set the management of the layers with an environment variable -- for example, to enable layer caching, run `export BUILDAH_LAYERS=true`.

While retaining intermediate layers consumes initial storage space, the true impact on your system is more nuanced. Keeping these layers allows for layer sharing across multiple different images, which can actually save significant space and bandwidth; if 10 images share the same base layers, those layers are stored and pulled only once.

However, there are a couple of distinct trade-offs to consider:

- **The** **downside of** **many** **layers**: Every additional layer adds complexity to the union filesystem. This can lead to slightly slower container startup times and increased overhead when the kernel merges the layers into a single view.
- **The** **downside of** **squashing**: While *squashing* layers into a single one can improve startup speed and permanently remove files deleted in upper layers (files that would otherwise remain hidden but present in a multi-layered image), it destroys deduplication. A squashed image cannot share its layers with others, meaning the total storage consumed on your host may actually increase if you maintain several similar images.

Ultimately, keeping intermediate layers is a balance: you trade a bit of filesystem complexity for massive gains in build speed and storage efficiency.

