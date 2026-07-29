# 13 {#Chapter_13.xhtml#h1_314 .chapterNumber}

# Docker Migration Tips and Tricks {#Chapter_13.xhtml#h1_315 .chapterTitle}

Every technology has a pioneer company, project, and product that, once
created and announced, has been a real game-changer that allowed its
base concepts to be spread.[]{.sentence-end} For containers, this was
Docker.

As we learned in *[Chapter 1](#Chapter_1.xhtml#h1_14){.chapref}*,
*Introduction to Container Technology*, Docker provided a new approach
and great ideas for leveraging existing technologies and creating brand
new ones.[]{.sentence-end} After a few years, it became the most used
technology for containers.

But as it usually happens for open source projects, the community and
the enterprise started looking for improvements, new architectures, and
different implementations.[]{.sentence-end} That\'s where Podman found a
place to grow and leverage the[]{#Chapter_13.xhtml#idx_08210be1
.index-entry index-entry="Open Container Initiative (OCI)"}
standardization that\'s offered by the **Open Container Initiative**
(**OCI**).

Docker was (and still is) the most used container
technology.[]{.sentence-end} For this reason, in this chapter, we are
going to provide some tips and tricks regarding handling the migration
process.[]{.sentence-end} We will be covering the following topics:

-   Migrating existing images and playing with a command\'s alias
-   Podman commands versus Docker commands
-   Using Docker Compose with Podman

# Technical requirements {#Chapter_13.xhtml#h1_316 .heading-1}

Before you proceed with this chapter\'s lecture and examples, you will
need a machine with a working Podman installation.[]{.sentence-end} As
we mentioned in *[Chapter 3](#Chapter_3.xhtml#h1_83){.chapref}*,
*Running* *the* *First Container*, all the examples in this book have
been executed on a Fedora 40 system or later, but can be reproduced on
your choice of **operating system** (**OS**).

Having a good understanding of the topics that were covered in *[Chapter
4](#Chapter_4.xhtml#h1_114){.chapref}*, *Managing Running Containers*,
*[Chapter 5](#Chapter_5.xhtml#h1_133){.chapref}*, *Implementing Storage
for the Container\'s Data*, and *[Chapter
9](#Chapter_9.xhtml#h1_219){.chapref}*, *Pushing Images to a Container
Registry*, will help you grasp the concepts that will be covered in this
chapter regarding containers.

# Migrating existing images and playing with a command\'s alias {#Chapter_13.xhtml#h1_317 .heading-1}

Podman[]{#Chapter_13.xhtml#idx_0cce4abb .index-entry
index-entry="CLI compatibility:with Docker"} has one great feature that
lets any previous Docker user easily adapt and switch to it---complete
**command-line interface** (**CLI**)
compatibility[]{#Chapter_13.xhtml#idx_0f935203 .index-entry
index-entry="images:migrating"} with Docker.

Let\'s demonstrate this CLI compatibility with Docker by creating a
shell command alias for the `docker`{.inlineCode} command:

``` {.programlisting .snippet-con}
# alias docker=podman
# docker
Error: missing command 'podman COMMAND'
Try 'podman --help' for more information.
```

As you can see, we have created a command alias that binds the
`podman`{.inlineCode} command to the `docker`{.inlineCode}
one.[]{.sentence-end} If we try to execute the `docker`{.inlineCode}
command after setting the alias, the output is returned from the
`podman`{.inlineCode} command instead.

As an alternative to the classic command alias, we can install the
`podman-docker`{.inlineCode} package, available in many distributions,
which installs a script named `docker`{.inlineCode} that emulates the
Docker CLI while executing Podman commands.[]{.sentence-end} The script
also creates links to the Podman `man`{.inlineCode} pages, so that users
writing `man `{.inlineCode}`docker`{.inlineCode} will get the output of
`man `{.inlineCode}`docker`{.inlineCode} instead.[]{.sentence-end} To
install the `podman-docker`{.inlineCode} package on Fedora, simply run
the following:

``` {.programlisting .snippet-con}
$ sudo dnf install podman-docker
```

The script installed by the package is really simple, as you can see in
the following code snippet:

``` {.programlisting .snippet-con}
$ cat /usr/bin/docker
#!/usr/bin/sh
[ -e /etc/containers/nodocker ] || [ -e "${XDG_CONFIG_HOME-$HOME/.config}/containers/nodocker" ] || \
echo "Emulate Docker CLI using podman. Create /etc/containers/nodocker to quiet msg." >&2
exec /usr/bin/podman "$@"
```

Let\'s try this []{#Chapter_13.xhtml#idx_56efb9a7 .index-entry
index-entry="CLI compatibility:with Docker"}out on the newly created
alias (or script when using the `podman-docker`{.inlineCode} package) by
running a container:

``` {.programlisting .snippet-con}
# docker version
Emulate Docker CLI using podman. Create /etc/containers/nodocker to quiet msg.
Client:        Podman Engine
Version:       5.4.2
API Version:   5.4.2
Go Version:    go1.22.12
Git Commit:    be85287fcf4590961614ee37be65eeb315e5d9ff
Built:         Wed Apr  2 00:00:00 2025
Build Origin:  Fedora Project
OS/Arch:       linux/amd64
# docker run --rm -it docker.io/wernight/funbox nyancat
```

We should see something very funny---a running cat, similar to the one
shown in the following screenshot:

<figure class="mediaobject">
<img src="images/B31467_13_1.png"
style="width:528.0px; height:295.68px;" alt="Image 1" />
</figure>

Figure 13.1 -- Funny output from running a test container

Let\'s test []{#Chapter_13.xhtml#idx_ad5ed419 .index-entry
index-entry="CLI compatibility:with Docker"}something more
interesting.[]{.sentence-end} Docker, for example, offers a tutorial
based on a container image exposing a web server:

``` {.programlisting .snippet-con}
# docker run -dp 80:80 docker.io/docker/getting-started
Trying to pull docker.io/docker/getting-started:latest...
Getting image source signatures
Copying blob 97518928ae5f done
Copying blob e0bae2ade5ec done
Copying blob a2402c2da473 done
Copying blob e362c27513c3 done
Copying blob a4e156412037 done
Copying blob 3f3577460f48 done
Copying blob 69465e074227 done
Copying blob eb65930377cd done
Copying config 26d80cd96d done
Writing manifest to image destination
Storing signatures
d44a2df41d76b3322e56971d45e92e75f4679e8b620198228fbd9cc00fe9578f
```

Here, we continued to use the `docker`{.inlineCode} alias command with
the option for running it by using a daemon,
`-`{.inlineCode}`d`{.inlineCode}, and the
[]{#Chapter_13.xhtml#idx_c78426cf .index-entry
index-entry="CLI compatibility:with Docker"}option for binding the HTTP
port, `-`{.inlineCode}`p`{.inlineCode}.

If everything worked correctly, then we can point our favorite web
browser to `http://`{.inlineCode}`localhost`{.inlineCode}:

<figure class="mediaobject">
<img src="images/B31467_13_2.png"
style="width:528.0px; height:248.9142857142857px;" alt="Image 2" />
</figure>

Figure 13.2 -- Docker tutorial home page

The first page of Docker Labs, **Getting Started**, specifies the
command that was just run.[]{.sentence-end} From the left column of the
page, we can continue with the tutorial.

Let\'s continue with the tutorial and double-check that the alias will
work properly at every stage.

The tutorial steps are very simple, and they can help you summarize the
knowledge that was shared in the previous chapters, from building a
container to using multiple container applications to create a dedicated
network.[]{.sentence-end} Please stop before the **Using** **Docker**
**Compose** section, as we will look at this in more detail shortly.

Don\'t forget that we are using an alias and that, under the hood,
Podman is working actively to let our containers work as expected,
ensuring Docker CLI compatibility.

But what about[]{#Chapter_13.xhtml#idx_afdc718e .index-entry
index-entry="CLI compatibility:with Docker"} container migration in the
case of swapping Docker in favor of Podman?

Well, a direct way to move existing containers from Docker to Podman
does not exist.[]{.sentence-end} It is recommended that you recreate the
containers with the respective container images and reattach any volumes
using Podman.

The container images can be exported using the
`docker`{.inlineCode}` export`{.inlineCode} command, which will create a
TAR archive file that can be imported into Podman via the
`podman`{.inlineCode}` import`{.inlineCode} command.[]{.sentence-end} If
you\'re using a container image registry, you can skip this.

To understand any limitations we may encounter when we\'re using
commands, examples, and resources written for Docker with our Podman
installation, let\'s compare various Podman and Docker commands.

# Podman commands versus Docker commands {#Chapter_13.xhtml#h1_318 .heading-1}

As we saw[]{#Chapter_13.xhtml#idx_f345ae1e .index-entry
index-entry="Podman commands:versus Docker commands"} in the previous
section, as well as in *[Chapter 2](#Chapter_2.xhtml#h1_52){.chapref}*,
*Comparing* *Podman* *and* *Docker*, the Podman CLI is based on the
Docker CLI.[]{.sentence-end} However, because Podman does not require a
runtime daemon to work, some of the Docker commands may not be directly
available, or they could require some workarounds.

The command list is exceptionally long, so the following table only
specifies a few:

  **Docker** **command**                                    **Podman** **command**
  --------------------------------------------------------- ---------------------------------------------------------
  `docker`{.inlineCode}                                     `podman`{.inlineCode}
  `docker`{.inlineCode}` `{.inlineCode}`ps`{.inlineCode}    `podman`{.inlineCode}` `{.inlineCode}`ps`{.inlineCode}
  `docker`{.inlineCode}` pull`{.inlineCode}                 `podman`{.inlineCode}` pull`{.inlineCode}
  `docker`{.inlineCode}` push`{.inlineCode}                 `podman`{.inlineCode}` push`{.inlineCode}
  `docker`{.inlineCode}` rename`{.inlineCode}               `podman`{.inlineCode}` rename`{.inlineCode}
  `docker`{.inlineCode}` restart`{.inlineCode}              `podman`{.inlineCode}` restart`{.inlineCode}
  `docker`{.inlineCode}` `{.inlineCode}`rm`{.inlineCode}    `podman`{.inlineCode}` `{.inlineCode}`rm`{.inlineCode}
  `docker`{.inlineCode}` `{.inlineCode}`rmi`{.inlineCode}   `podman`{.inlineCode}` `{.inlineCode}`rmi`{.inlineCode}
  `docker`{.inlineCode}` run`{.inlineCode}                  `podman`{.inlineCode}` run`{.inlineCode}

As you can see, the command\'s name is the same when comparing the
`docker`{.inlineCode} command with the `podman`{.inlineCode}
command.[]{.sentence-end} However, even though the name is the same, due
to architectural differences between Podman and Docker, some features or
behaviors could be different.

## Behavioral differences between Podman and Docker {#Chapter_13.xhtml#h2_319 .heading-2}

The following []{#Chapter_13.xhtml#idx_1ac89157 .index-entry
index-entry="Podman commands:versus Docker commands"}commands were
intentionally implemented in another way by the Podman development team:

-   `podman`{.inlineCode}` volume create`{.inlineCode}: This command
    will fail if the volume already exists.[]{.sentence-end} In Docker,
    this command is idempotent, which means that if a volume already
    exists with the same name, then Docker will just skip this
    instruction.[]{.sentence-end} The actual behavior of Docker does not
    match the implementations for the other commands.
-   `podman`{.inlineCode}` run -v /`{.inlineCode}`tmp`{.inlineCode}`/`{.inlineCode}`noexist`{.inlineCode}`:/`{.inlineCode}`tmp`{.inlineCode}:
    This command will fail if the source volume path does not
    exist.[]{.sentence-end} Instead, Docker will create the folder if it
    does not exist.[]{.sentence-end} Again, the Podman development team
    considered this a bug and changed it.
-   `podman`{.inlineCode}` run --restart`{.inlineCode}: In older
    releases, the `restart`{.inlineCode} option in Podman did not
    persist after a system reboot.[]{.sentence-end} Modern Fedora and
    RHEL systems include a helper service that *bridges* this
    gap.[]{.sentence-end} By running
    `sudo`{.inlineCode}` `{.inlineCode}`systemctl`{.inlineCode}` enable --now `{.inlineCode}`podman-restart.service`{.inlineCode},
    Podman will scan for containers with `always`{.inlineCode} or
    `unless-stopped`{.inlineCode} policies during the boot process and
    start them automatically.

In the next section, we\'ll see which commands are missing from Podman
that exist in Docker.

## Missing commands in Podman {#Chapter_13.xhtml#h2_320 .heading-2}

The following[]{#Chapter_13.xhtml#idx_97d21e88 .index-entry
index-entry="Podman:missing commands"} table shows a non-comprehensive
list of Docker commands that, at the time of writing, don\'t have
equivalents in Podman:

  **Missing** **command**                       **Description**
  --------------------------------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  `docker`{.inlineCode}` plugin`{.inlineCode}   Podman does not support plugins at the time of writing.[]{.sentence-end} Its development team recommends using alternative OCI runtimes or OCI runtime hooks to alter its behavior.
  `docker`{.inlineCode}` swarm`{.inlineCode}    It is important to clarify that Podman does not support Docker Swarm.[]{.sentence-end} For users coming from the Docker ecosystem who require multi-node clustering and orchestration, the Podman project recommends moving toward Kubernetes.[]{.sentence-end} In a production Kubernetes environment, you typically won\'t find the Podman CLI running the cluster.[]{.sentence-end} Instead, Kubernetes uses a specialized, *bare-bones* runtime such as CRI-O.
  `docker`{.inlineCode}` trust`{.inlineCode}    This Docker command has been implemented in the `podman`{.inlineCode}` image trust`{.inlineCode} command.

Now, let\'s see which []{#Chapter_13.xhtml#idx_1b5c0a24 .index-entry
index-entry="Podman:missing commands"}commands are missing from Docker.

## Missing commands in Docker {#Chapter_13.xhtml#h2_321 .heading-2}

Similar to []{#Chapter_13.xhtml#idx_94db22db .index-entry
index-entry="Docker:missing commands"}how Podman is missing some Docker
commands, Docker is missing some Podman commands.

The following families of commands in Podman don\'t have respective ones
in Docker:

-   `podman`{.inlineCode}` generate`{.inlineCode}: This command can be
    used to create a structured output (such as a YAML file) for a
    container, pod, or volume.
-   `podman`{.inlineCode}` `{.inlineCode}`healthcheck`{.inlineCode}:
    This command provides you with a set of subcommands that you can use
    to manage container health checks.
-   `podman`{.inlineCode}` machine`{.inlineCode}: This command lists a
    set of subcommands for managing Podman\'s virtual machine on macOS
    and Windows (it also works on Linux, though you can already run
    containers natively, so it\'s not very useful there).
-   `podman`{.inlineCode}` mount`{.inlineCode}: This command mounts the
    container\'s root filesystem in a location that can be accessed by
    the host.
-   `podman`{.inlineCode}` play`{.inlineCode}: This command creates
    containers, pods, or volumes based on the input from a structured
    file (such as YAML) .
-   `podman`{.inlineCode}` pod`{.inlineCode}: This provides a set of
    subcommands for managing pods or groups of containers.
-   `podman`{.inlineCode}` `{.inlineCode}`unmount`{.inlineCode}: This
    command unmounts a working container\'s root filesystem.
-   `podman`{.inlineCode}` `{.inlineCode}`unshare`{.inlineCode}: This
    command launches a process in a new user namespace (rootless
    containers).
-   `podman`{.inlineCode}` `{.inlineCode}`untag`{.inlineCode}: This
    command removes one or more stored images.
-   `podman`{.inlineCode}` `{.inlineCode}`kube`{.inlineCode}: A critical
    addition for modern workflows, this command allows you to interact
    with []{#Chapter_13.xhtml#idx_0f6bbd8b .index-entry
    index-entry="Docker:missing commands"}Kubernetes
    YAML.[]{.sentence-end} You can use
    `podman`{.inlineCode}` `{.inlineCode}`kube`{.inlineCode}` play`{.inlineCode}
    to launch a local pod based on a K8s manifest, or
    `podman`{.inlineCode}` `{.inlineCode}`kube`{.inlineCode}` generate`{.inlineCode}
    to export your local container configuration for deployment into a
    cluster.
-   `podman`{.inlineCode}` volume export`{.inlineCode} and
    `podman`{.inlineCode}` volume import`{.inlineCode}: Unlike Docker,
    which typically requires manual file copying or custom scripts to
    move volume data, Podman provides native subcommands to compress a
    volume\'s contents into a tarball and move it between different
    systems or backups.

Of course, if a command is missing, this does not mean that the feature
is missing in Docker.

Another useful feature that\'s available in Docker is
Compose.[]{.sentence-end} We\'ll learn how to use it in Podman in the
next section.

# Using Docker Compose with Podman {#Chapter_13.xhtml#h1_322 .heading-1}

When it was []{#Chapter_13.xhtml#idx_72f37f81 .index-entry
index-entry="Docker Compose:using, with Podman"}first released, Docker
quickly gained consensus thanks to its intuitive approach to container
management.[]{.sentence-end} Along with the main container engine
solution, another great feature was introduced to help users orchestrate
multiple containers on a single host: **Docker** **Compose**.

The idea behind Compose is quite simple---it\'s a tool that can be used
to orchestrate multi-container applications that are supposed to
interact together on a single host and configured with a declarative
file in YAML format.[]{.sentence-end} All the applications that are
executed in a Compose stack are defined as services that can communicate
with the other containers in the stack with transparent name resolution.

The configuration file is named `docker-compose.yaml`{.inlineCode} and
has a simple syntax where one or more **services** and related
**volumes** are created and started.

Development teams can benefit from the stack\'s automation to quickly
test applications on a single host.[]{.sentence-end} However, if we need
to run our application on a production-like, multi-node environment, the
best approach is to adopt a clustered orchestration solution such as
Kubernetes.

When Podman []{#Chapter_13.xhtml#idx_12fa872f .index-entry
index-entry="Docker Compose:using, with Podman"}was first released, its
primary mission was to achieve OCI compatibility and feature parity with
the Docker CLI.[]{.sentence-end} However, one significant gap remained:
support for Docker Compose.[]{.sentence-end} In the early versions of
Podman, there was no background daemon or REST API for the original
`docker`{.inlineCode}`-compose`{.inlineCode} utility to communicate
with, making it impossible to run multi-container YAML definitions
directly.

To solve this, the community launched the
`podman`{.inlineCode}`-compose`{.inlineCode} project.[]{.sentence-end}
It is important to note that
`podman`{.inlineCode}`-compose`{.inlineCode} is an independent,
community-maintained tool---it was not created nor is it officially
maintained by the core Podman development team.[]{.sentence-end} It was
specifically designed to bridge the gap before Podman 3.0 by acting as a
Python-based wrapper that translated Compose YAML files directly into
`podman`{.inlineCode} commands.

The landscape changed significantly with the release of Podman 3.0,
which introduced a native, Docker-compatible Unix socket and a robust
REST API.[]{.sentence-end} This update allowed the original, official
`docker`{.inlineCode}`-compose`{.inlineCode} command (written in Go) to
*talk* directly to Podman as if it were the Docker daemon.

In this section, we will learn how to configure Podman to orchestrate
multiple containers with `docker`{.inlineCode}`-compose`{.inlineCode} to
provide full compatibility to users migrating from Docker to
Podman.[]{.sentence-end} In the next subsection, we\'ll look at an
example of using `podman`{.inlineCode}`-compose`{.inlineCode} to
leverage rootless container orchestration.

Before we dig into setting up Podman, let\'s look at a few basic
examples of Compose files to understand how they work.

## Docker Compose quick start {#Chapter_13.xhtml#h2_323 .heading-2}

Compose[]{#Chapter_13.xhtml#idx_404d13fb .index-entry
index-entry="Docker Compose:quick start"} files can be used to declare
one or multiple containers being executed inside a common stack and also
to define build instructions for custom applications.[]{.sentence-end}
The advantage of this approach is that you can fully automate the entire
application stack, including frontends, backends, and persistence
services such as databases or in-memory caches.

 note
**Important** **note**

The purpose of this section is to provide a quick overview of Compose
files to help you understand how Podman can handle them.

For a detailed list of the latest Compose specification, please refer to
the following URL:
[[https://docs.docker.com/reference/compose-file/]{.url}](https://docs.docker.com/reference/compose-file/){style="text-decoration: none;"}.

A more extensive list of Compose examples can be found in the Docker
Awesome Compose project at
[[https://github.com/docker/awesome-compose]{.url}](https://github.com/docker/awesome-compose){style="text-decoration: none;"}.


The following is a minimal configuration file that defines a single
container running the Docker registry
(`Chapter13`{.inlineCode}`/registry/`{.inlineCode}`docker-compose.yaml`{.inlineCode}):

``` {.programlisting .snippet-code}
services:
  registry:
    ports:
      - "5000:5000"
    volumes:
      - registry_volume:/var/lib/registry
    image: docker.io/library/registry
volumes:
  registry_volume: {}
```

The preceding example can be seen as a more structured and declarative
way to define the execution parameters for a container.[]{.sentence-end}
However, the real value of Docker Compose is its orchestration stacks,
which are made up of multiple containers in a single instance.

The following example is even more interesting and shows a configuration
file for a WordPress application that uses
a[]{#Chapter_13.xhtml#idx_7f0c0ae2 .index-entry
index-entry="Docker Compose:quick start"} MySQL database as its backend
(`Chapter13`{.inlineCode}`/`{.inlineCode}`wordpress`{.inlineCode}`/`{.inlineCode}`docker-compose.yaml`{.inlineCode}):

``` {.programlisting .snippet-code}
services:
  db:
    image: docker.io/library/mysql:latest
    command: '--default-authentication-plugin=mysql_native_password'
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=wordpressroot
      - MYSQL_DATABASE=wordpress
      - MYSQL_USER=wordpress
      - MYSQL_PASSWORD=wordpress
    expose:
      - 3306
      - 33060
  wordpress:
    image: docker.io/library/wordpress:latest
    ports:
      - 8080:80
    restart: always
    environment:
      - WORDPRESS_DB_HOST=db
      - WORDPRESS_DB_USER=wordpress
      - WORDPRESS_DB_PASSWORD=wordpress
      - WORDPRESS_DB_NAME=wordpress
volumes:
  db_data:
```

Here, we can see the two main YAML objects---`services`{.inlineCode} and
`volumes`{.inlineCode}.[]{.sentence-end} Under the
`services`{.inlineCode} part of the code, we have two
applications---`db`{.inlineCode} and
`wordpress`{.inlineCode}.[]{.sentence-end} These have been highlighted
for clarity.

In the `services`{.inlineCode} list, there\'s a set
[]{#Chapter_13.xhtml#idx_25be74bc .index-entry
index-entry="Docker Compose:quick start"}of configuration values that
define the container\'s behavior.[]{.sentence-end} These are as follows:

-   `image`{.inlineCode}: The image that\'s used by the container.
-   `command`{.inlineCode}: Additional commands to be passed to the
    container\'s entry point.
-   `v`{.inlineCode}`olumes`{.inlineCode}: The list of volumes to be
    mounted in the container, along with their associated mount
    points.[]{.sentence-end} Along with new dedicated volumes, existing
    directories in the host can be bind-mounted on container mount
    points.
-   `restart`{.inlineCode}: Container restart options in case an error
    occurs.
-   `expose`{.inlineCode}: The list of ports to be exposed by the
    container.
-   `ports`{.inlineCode}: The list of port mappings between the
    container and the host.[]{.sentence-end}
-   `environment`{.inlineCode}: The list of environment variables to be
    created in the container.[]{.sentence-end} In this example,
    `WORDPRESS_DB_HOST`{.inlineCode}, `WORDPRESS_DB_USER`{.inlineCode},
    `WORDPRESS_DB_PASSWORD`{.inlineCode}, and
    `WORDPRESS_DB_NAME`{.inlineCode} are injected into the WordPress
    container to provide connection parameters to the database.

Together with the `services`{.inlineCode} declaration, we have a list of
volumes that are managed by Compose.[]{.sentence-end} The engine can
create these volumes in the Compose process or use existing volumes that
have been labeled as `external`{.inlineCode}.

The third and final example is a Compose file that builds a minimal REST
API application that\'s been written in Go, which writes and retrieves
data to a Redis in-memory store
(`Chapter13`{.inlineCode}`/`{.inlineCode}`golang-redis`{.inlineCode}`/`{.inlineCode}`docker-compose.yaml`{.inlineCode}):

``` {.programlisting .snippet-code}
services:
  web:
    build:
      context: ./app
      labels:
        - "com.example.description=Golang Redis App"
    ports:
      - "8080:8080"
    environment:
      - REDIS_HOST=redis
    depends_on:
      - redis
  redis:
    image: docker.io/library/redis
    deploy:
      replicas: 1
```

In this example, we []{#Chapter_13.xhtml#idx_add127cc .index-entry
index-entry="Docker Compose:quick start"}have new elements that deserve
attention:

-   `build`{.inlineCode}: Defines the image to be built and also applies
    custom labels to the build.
-   `context`{.inlineCode}: Holds the path for the
    build.[]{.sentence-end} In this example, the
    `./`{.inlineCode}`app`{.inlineCode} folder contains all the source
    code files and the Dockerfile for building the image.
-   `labels`{.inlineCode}: Holds a set of labels that are passed as
    strings in the build process.
-   `depends_on`{.inlineCode}: Specifies, for the web service, the other
    services that are considered dependencies---in this case, the
    `redis`{.inlineCode} service.
-   `environment`{.inlineCode}: Defines the name of the
    `redis`{.inlineCode} service that\'s used by the web app.
-   `deploy`{.inlineCode}: An object in the `redis`{.inlineCode} service
    that lets us define custom configuration parameters, such as the
    number of container replicas.

To bring up Compose applications with Docker, we can run the following
command from the Compose file\'s folder:

``` {.programlisting .snippet-con}
$ docker-compose up
```

This command creates all the stack and related volumes while printing
the output to `stdout`{.inlineCode}.

To run in detached mode, simply add the `-d`{.inlineCode} option to the
command:

``` {.programlisting .snippet-con}
$ docker-compose up -d
```

The following command builds the necessary images and starts the stack:

``` {.programlisting .snippet-con}
$ docker-compose up --build
```

Alternatively, the `docker`{.inlineCode}`-compose build`{.inlineCode}
command can be used to build the applications without starting them.

To shut[]{#Chapter_13.xhtml#idx_0249f64c .index-entry
index-entry="Docker Compose:quick start"} down a stack running in the
foreground, simply hit the *Ctrl* + *C* keyboard
combination.[]{.sentence-end} Instead, to shut down a detached
application, run the following command:

``` {.programlisting .snippet-con}
$ docker-compose down
```

To kill an unresponsive container, we can use the
`docker`{.inlineCode}`-compose kill`{.inlineCode} command:

``` {.programlisting .snippet-con}
$ docker-compose kill [SERVICE]
```

This command also supports multiple signals with the
`-s SIGNAL`{.inlineCode} option.

Now that we\'ve covered the basic concepts surrounding Docker Compose,
let\'s learn how to configure Podman to run Compose files.

## Configuring Podman to interact with docker-compose {#Chapter_13.xhtml#h2_324 .heading-2}

To[]{#Chapter_13.xhtml#idx_291e9072 .index-entry
index-entry="Docker Compose:Podman, configuring for interaction"}
support Compose, Podman needs to expose its REST API service through a
local Unix socket.[]{.sentence-end} This service supports both
Docker-compatible APIs and the native Libpod APIs.

On a Fedora distribution, the
`docker`{.inlineCode}`-compose`{.inlineCode} package (which provides
Docker Compose binaries) and the `podman-docker`{.inlineCode} package
(which provides aliasing to the `docker`{.inlineCode} command) must be
installed using the following command:

``` {.programlisting .snippet-con}
$ sudo dnf install docker-compose podman-docker
```

 note
**Important** **note**

The `docker`{.inlineCode}`-compose`{.inlineCode} package, when installed
on a Fedora 42 system, installs version 2.37.3 at the time of
writing.[]{.sentence-end} The latest version, 2, was completely
rewritten in Go, while version 1.x was written in
Python.[]{.sentence-end} The new implementation provides a significant
performance improvement and can be downloaded from the GitHub release
page at
[[https://github.com/docker/compose/releases]{.url}](https://github.com/docker/compose/releases){style="text-decoration: none;"}.


After installing the packages, we can enable and start the
`systemd`{.inlineCode} unit that manages the Unix socket service:

``` {.programlisting .snippet-con}
$ sudo systemctl enable --now podman.socket
```

This[]{#Chapter_13.xhtml#idx_523bfb68 .index-entry
index-entry="Docker Compose:Podman, configuring for interaction"}
command starts a socket that\'s listening on
`/run/`{.inlineCode}`podman`{.inlineCode}`/`{.inlineCode}`podman.sock`{.inlineCode}.

Note that the native `docker`{.inlineCode}`-compose`{.inlineCode}
command looks for a socket file in the
`/run/`{.inlineCode}`docker.sock`{.inlineCode} path by
default.[]{.sentence-end} For this reason, the
`podman-docker`{.inlineCode} package creates a symbolic link on the same
path that points to
`/run/`{.inlineCode}`podman`{.inlineCode}`/`{.inlineCode}`podman.sock`{.inlineCode},
as shown in the following example:

``` {.programlisting .snippet-con}
# ls -al /run/docker.sock
lrwxrwxrwx. 1 root root 23 Feb  3 21:54 /var/run/docker.sock -> /run/podman/podman.sock
```

The Unix socket that\'s exposed by Podman can be accessed by a process
with root privileges only.[]{.sentence-end} It is possible to stretch
the security restrictions by opening access to the file to all the users
in the system or by allowing custom ACLs for a custom
group.[]{.sentence-end} Later in this chapter, we will see that rootless
container stacks can be executed with
`podman`{.inlineCode}`-compose`{.inlineCode}.

For the sake of simplicity, in the next subsection, you\'ll learn how to
run `docker`{.inlineCode}`-compose`{.inlineCode} commands with Podman in
rootful mode.

## Running Compose workloads with Podman and docker-compose {#Chapter_13.xhtml#h2_325 .heading-2}

To []{#Chapter_13.xhtml#idx_1663314e .index-entry
index-entry="Docker Compose:workloads, running with Podman and docker-compose"}help
you learn how to operate `docker`{.inlineCode}`-compose`{.inlineCode}
and create orchestrated multi-container deployments on our host, we will
reuse the previous example of the Go REST API with a Redis in-memory
store.

We have already inspected the `docker-compose.yaml`{.inlineCode} file,
which builds the web application and deploys one instance of the Redis
container.

Let\'s inspect[]{#Chapter_13.xhtml#idx_9266d05c .index-entry
index-entry="Docker Compose:workloads, running with Podman and docker-compose"}
the Dockerfile that\'s used to build the application
(`Chapter13`{.inlineCode}`/`{.inlineCode}`golang-redis`{.inlineCode}`/`{.inlineCode}`Dockerfile`{.inlineCode}):

``` {.programlisting .snippet-code}
FROM docker.io/library/golang AS builder

# Copy files for build
RUN mkdir -p /go/src/golang-redis
COPY go.mod main.go /go/src/golang-redis

# Set the working directory
WORKDIR /go/src/golang-redis

# Download dependencies
RUN go get -d -v ./...

# Install the package
RUN go build -v

# Runtime image
FROM registry.access.redhat.com/ubi9/ubi-minimal:latest as bin
COPY --from=builder /go/src/golang-redis/golang-redis /usr/local/bin
COPY entrypoint.sh /

EXPOSE 8080

ENTRYPOINT ["/entrypoint.sh"]
```

Here, we can see that the Go application is compiled in a multi-stage
build and that the Go binary is copied inside a UBI-Minimal image.

The web frontend is minimal---it listens to port
`8080`{.inlineCode}/`tcp`{.inlineCode} and only implements two
endpoints---an HTTP `POST`{.inlineCode} method and an HTTP
`GET`{.inlineCode} method to allow clients to upload and retrieve a JSON
object that contains the name, email, and ID of a user.[]{.sentence-end}
The JSON object is stored inside the Redis database.

 note
**Important** **note**

If you\'re curious, the source code for the Go server is available in
the
`Chapter13`{.inlineCode}`/`{.inlineCode}`golang-redis`{.inlineCode}`/app/`{.inlineCode}`main.go`{.inlineCode}
file.[]{.sentence-end} It isn\'t presented in this book for the sake of
space and readability.


To build and []{#Chapter_13.xhtml#idx_78f75913 .index-entry
index-entry="Docker Compose:workloads, running with Podman and docker-compose"}run
the application, we must change to the project directory and run the
`docker`{.inlineCode}`-compose up`{.inlineCode} command:

``` {.programlisting .snippet-con}
# cd Chapter13/golang-redis
# docker-compose up --build -d
[...omitted output...]
Successfully tagged localhost/golang-redis_web:latest
6b330224010ed611baba11fc2d66b9e4cfc991312f5166b47b5fcd07357c6325
Successfully built 6b330224010ed611baba11fc2d66b9e4cfc991312f5166b47b5fcd07357c6325
Creating golang-redis_redis_1 ... done
Creating golang-redis_web_1   ... done
```

Here, we can see that `docker`{.inlineCode}`-compose`{.inlineCode}
created two containers, whose names always follow the
`<`{.inlineCode}`project`{.inlineCode}`_name`{.inlineCode}`>`{.inlineCode}`_`{.inlineCode}`<`{.inlineCode}`service_name`{.inlineCode}`>`{.inlineCode}`_`{.inlineCode}`<`{.inlineCode}`instance_count`{.inlineCode}`>`{.inlineCode}
pattern.

The instance count varies when there is more than one replica in the
service deployment.[]{.sentence-end} For example, if we set the
`redis`{.inlineCode} service value of replicas to `3`{.inlineCode}, we
will get 3 container instances of the service.

We can inspect the running containers with the usual
`podman`{.inlineCode}` `{.inlineCode}`ps`{.inlineCode} command:

``` {.programlisting .snippet-con}
# podman ps
CONTAINER ID  IMAGE                              COMMAND       CREATED         STATUS             PORTS                   NAMES
4a5421c9e7cd  docker.io/library/redis:latest     redis-server  20 seconds ago  Up 20 seconds ago                          golang-redis_redis_1
8a465d4724ab  localhost/golang-redis_web:latest                20 seconds ago  Up 20 seconds ago  0.0.0.0:8080->8080/tcp  golang-redis_web_1
```

One of the []{#Chapter_13.xhtml#idx_c1665f10 .index-entry
index-entry="Docker Compose:workloads, running with Podman and docker-compose"}more
interesting aspects is that the service names are automatically
resolved.

When a Compose stack is created, Podman creates a new network, named
with the
`<`{.inlineCode}`project`{.inlineCode}`_`{.inlineCode}`name`{.inlineCode}`>`{.inlineCode}`_default`{.inlineCode}
pattern.

The new network instantiates an
`aardvark-`{.inlineCode}`dns`{.inlineCode} process and resolves the
containers\' IPs to names that have been created after the service
names.

The `aardvark-`{.inlineCode}`dns`{.inlineCode} service can be found
using the `ps`{.inlineCode} command and filtered with
`grep`{.inlineCode}:

``` {.programlisting .snippet-con}
ps aux | grep aardvark-dns
gbsalin+  318577  0.0  0.0 1493272 2944 ?        Ssl  14:01   0:00 /usr/libexec/podman/aardvark-dns --config /run/user/1000/containers/networks/aardvark-dns -p 53 run
gbsalin+  321031  0.0  0.0 231256  2448 pts/2    S+   14:24   0:00 grep --color=auto aardvark-dns
```

The
`/run/user/`{.inlineCode}`<`{.inlineCode}`USER`{.inlineCode}`_ID`{.inlineCode}`>`{.inlineCode}`/containers/networks/aardvark-`{.inlineCode}`dns`{.inlineCode}`/`{.inlineCode}
directory holds the instance\'s configuration.[]{.sentence-end} Inside
the `golang-redis_default`{.inlineCode} file, we can find the mappings
between the service names and the allocated container IPs:

``` {.programlisting .snippet-con}
cat golang-redis_default
10.89.5.1
843bfd9cf8f766843340e228033ee89d61bb9916b398c2537bc3c600fe6a74e8 10.89.5.2  golang-redis-redis-1,golang-redis-redis-1,redis,843bfd9cf8f7
776daece0e231f18b349fcc68deccee41e307355c3b4e4db279243fe6a081dcc 10.89.5.3  golang-redis-redis-2,golang-redis-redis-2,redis,776daece0e23
adc4344449224c1c62c75f39121dea87ad6be5e308e794e97acffecdafd624cb 10.89.5.4  golang-redis-redis-3,golang-redis-redis-3,redis,adc434444922
```

This means that a process inside a container can resolve a service name
with a standard DNS query.

When we have multiple container replicas in a service, the resulting
resolution that\'s delivered by
`aardvark-`{.inlineCode}`dns`{.inlineCode} is similar to a **round-robin
DNS**, a[]{#Chapter_13.xhtml#idx_4c73708a .index-entry
index-entry="round-robin DNS"} simple and minimalistic kind of load
balancing that iterates multiple DNS records that are resolved by the
same name.[]{.sentence-end} When a process
calls[]{#Chapter_13.xhtml#idx_3d4aa6d3 .index-entry
index-entry="Docker Compose:workloads, running with Podman and docker-compose"}
a service (the `db`{.inlineCode} service, for example), it will be
resolved to as many different IPs as there are service replicas.

Let\'s go back to the `docker-compose.yaml`{.inlineCode}
file.[]{.sentence-end} In the `environment`{.inlineCode} section of the
`web`{.inlineCode} service configuration, we have the following
variable:

``` {.programlisting .snippet-code}
   environment:
      - REDIS_HOST=redis
```

This variable is injected into the running container and represents the
name of the `redis`{.inlineCode} service.[]{.sentence-end} It is used by
the Go application to create the connection string to Redis and
initialize the connection.[]{.sentence-end} When we\'re using a
DNS-resolved service name, the container name and IP address of the
`redis`{.inlineCode} service are completely irrelevant to our Go
application.

We can use the `docker`{.inlineCode}`-compose exec`{.inlineCode} command
to verify that the variable was correctly injected inside the containers
running as the `web`{.inlineCode} service in the stack:

``` {.programlisting .snippet-con}
# docker-compose exec web env
Emulate Docker CLI using podman. Create /etc/containers/nodocker to quiet msg.
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
TERM=xterm
container=oci
REDIS_HOST=redis
HOME=/root
```

The `env`{.inlineCode} command outputs the full list of environment
variables in the container.[]{.sentence-end} This allows us to verify
that the `REDIS_HOST`{.inlineCode} variable was created correctly.

 note
**Important** **note**

Storing configurations such as connection strings in a database as
constants in the application code is an anti-pattern in general,
especially for modern cloud-native applications.[]{.sentence-end} The
correct approach is to guarantee a strict separation between the
application logic and the configuration parameters.

Configurations can be stored as environment variables or inside
config/secret files that are injected at runtime in the container that
runs the application.

These practices are well-defined in the **Twelve-Factor App** pattern
specification, whose URL can be found in the *Further reading* section.


It is finally time to[]{#Chapter_13.xhtml#idx_4556053a .index-entry
index-entry="Docker Compose:workloads, running with Podman and docker-compose"}
test the application by posting a couple of JSON objects and retrieving
one of them with the `curl`{.inlineCode} command:

``` {.programlisting .snippet-con}
$ curl -X POST -d \
'{"name":"jim", "email":"jim@example.com", "id":"0001"}' \
localhost:8080
$ curl -X POST -d \
'{"name":"anna", "email":"anna@example.com", "id":"0002"}' \
localhost:8080
```

The `web`{.inlineCode} container was successfully written to the Redis
backend, which we can see by running the
`docker`{.inlineCode}`-compose logs`{.inlineCode} command:

``` {.programlisting .snippet-con}
# docker-compose logs web
[...omitted output...]
2022/02/06 00:58:06 Storing data:  {"name":"jim","email":"jim@example.com","id":"0001"}
2022/02/06 00:58:10 Storing data:  {"name":"anna","email":"anna@example.com","id":"0002"}
```

The preceding command captures the logs of all the containers behind the
`web`{.inlineCode} service.

Finally, we can retrieve the result.[]{.sentence-end} The web
application reads back the object from the Redis database by looking at
its `id`{.inlineCode} value:

``` {.programlisting .snippet-con}
$ curl -X GET -d '{"id": "0001"}' localhost:8080
{"name":"jim","email":"jim@example.com","id":"0001"}
```

To shut down our application, we can simply use the
`docker`{.inlineCode}`-compose down`{.inlineCode} command:

``` {.programlisting .snippet-con}
# docker-compose down
```

This[]{#Chapter_13.xhtml#idx_11166c34 .index-entry
index-entry="Docker Compose:workloads, running with Podman and docker-compose"}
command destroys the containers and their associated resources,
including the custom network, but not volumes.[]{.sentence-end} To
remove volumes, you must add the `-v`{.inlineCode} option to the end of
the command.

The `docker`{.inlineCode}`-compose`{.inlineCode} utility is a great
companion for building and deploying on a single host with
Podman.[]{.sentence-end} However, in the next chapter, we will learn
about some other useful solutions that will let us generate and execute
Kubernetes Pod and Service resources, as well as containers that are
executed by `s`{.inlineCode}`ystemd`{.inlineCode}
units.[]{.sentence-end} Before moving on, let\'s inspect the alternative
`podman`{.inlineCode}`-compose`{.inlineCode} tool, which provides
support for rootless containers.

## Using podman-compose {#Chapter_13.xhtml#h2_326 .heading-2}

The `podman`{.inlineCode}`-compose`{.inlineCode} project
[]{#Chapter_13.xhtml#idx_7f1a7ebb .index-entry
index-entry="Docker Compose:podman-compose, using"}started way before
[]{#Chapter_13.xhtml#idx_7edb86db .index-entry
index-entry="podman-compose project"}version 3.0 of Podman to provide a
compatibility layer for users who needed to orchestrate containers with
Compose files.[]{.sentence-end} In this subsection, we will look at an
example of using `podman`{.inlineCode}`-compose`{.inlineCode} on Fedora.

The `podman`{.inlineCode}`-compose`{.inlineCode} tool\'s CLI is written
in Python.[]{.sentence-end} The package can be installed with
`dnf`{.inlineCode} or by getting the latest release from the respective
GitHub repository (you can find the direct link in the *Further reading*
section):

``` {.programlisting .snippet-con}
$ sudo dnf install -y podman-compose
```

Alternatively, it can be installed with Python\'s package manager,
`pip3`{.inlineCode}, which supports a broader set of operating systems
and distributions:

``` {.programlisting .snippet-con}
$ pip3 install podman-compose
```

Now, we can run the same Compose stacks from the previous examples with
the advantage of the rootless approach that\'s provided by
`podman`{.inlineCode}`-compose`{.inlineCode}.

The following are all the available commands that are compatible with
`docker`{.inlineCode}`-compose`{.inlineCode}, along
[]{#Chapter_13.xhtml#idx_5a0c7bbd .index-entry
index-entry="podman-compose project"}with their
[]{#Chapter_13.xhtml#idx_47ffdea7 .index-entry
index-entry="Docker Compose:podman-compose, using"}descriptions and some
minor changes that are made by the output of the
`podman`{.inlineCode}`-compose help`{.inlineCode} command:

-   `help`{.inlineCode}: Shows the tool\'s help
-   `version`{.inlineCode}: Shows the command\'s version
-   `systemd`{.inlineCode}: Creates `systemd`{.inlineCode} unit files
    and registers its Compose tasks
-   `pull`{.inlineCode}: Pulls the stack images
-   `push`{.inlineCode}: Pushes the stack images
-   `build`{.inlineCode}: Builds the stack images
-   `up`{.inlineCode}: Creates and starts the entire stack or some of
    its services
-   `down`{.inlineCode}: Tears down the entire stack
-   `ps`{.inlineCode}: Show the status of running containers
-   `run`{.inlineCode}: Creates a container similar to a service to run
    a one-off command
-   `exec`{.inlineCode}: Executes a certain command in a running
    container
-   `start`{.inlineCode}: Starts specific services
-   `stop`{.inlineCode}: Stops specific services
-   `restart`{.inlineCode}: Restarts specific services
-   `logs`{.inlineCode}: Shows logs from services
-   `config`{.inlineCode}: Displays the Compose file
-   `port`{.inlineCode}: Shows the public port for a port binding
-   `pause`{.inlineCode}: Pauses all running containers
-   `unpause`{.inlineCode}: Unpauses the previously paused containers
-   `kill`{.inlineCode}: Kills one or more containers with a specific
    signal
-   `stats`{.inlineCode}: Shows usage stats (CPU, memory, network I/O,
    block I/O, and PIDs)
-   `images`{.inlineCode}: Lists the images used by the running
    containers

The following command creates a stack from a directory containing the
necessary configurations and the `docker-compose.yaml`{.inlineCode}
file:

``` {.programlisting .snippet-con}
$ podman-compose up
```

The[]{#Chapter_13.xhtml#idx_ca522fa7 .index-entry
index-entry="Docker Compose:podman-compose, using"} command\'s output is
also very similar to the output provided by
`docker`{.inlineCode}`-compose`{.inlineCode}.

To build and []{#Chapter_13.xhtml#idx_14b0d0c0 .index-entry
index-entry="podman-compose project"}contextually run the services stack
as we did in the previous `golang-redis`{.inlineCode} example, use this:

``` {.programlisting .snippet-con}
$ podman-compose up --build -d
```

To shut down the stack, simply run the following command:

``` {.programlisting .snippet-con}
$ podman-compose down
```

The `podman`{.inlineCode}`-compose `{.inlineCode}`systemd`{.inlineCode}
command provides an interesting way to interact with
`systemd`{.inlineCode} and register our Compose stacks as
`systemd`{.inlineCode} units.[]{.sentence-end} The following example
creates a `systemd`{.inlineCode} unit and registers it:

``` {.programlisting .snippet-con}
sudo podman-compose systemd -a create-unit
Sudo podman-compose systemd -a register
```

The first command creates a unit such as the following under the
`/`{.inlineCode}`etc`{.inlineCode}`/`{.inlineCode}`systemd`{.inlineCode}`/user/`{.inlineCode}`podman`{.inlineCode}`-`{.inlineCode}`compose@.service`{.inlineCode}
path:

``` {.programlisting .snippet-con}
# /etc/systemd/user/podman-compose@.service

[Unit]
Description=%i rootless pod (podman-compose)

[Service]
Type=simple
EnvironmentFile=%h/.config/containers/compose/projects/%i.env
ExecStartPre=-/usr/bin/podman-compose up --no-start
ExecStartPre=/usr/bin/podman pod start pod_%i
ExecStart=/usr/bin/podman-compose wait
ExecStop=/usr/bin/podman pod stop pod_%i

[Install]
WantedBy=default.target
```

From []{#Chapter_13.xhtml#idx_2430f0f3 .index-entry
index-entry="Docker Compose:podman-compose, using"}now on, it is
possible to start, stop, or inspect our service by recalling the
`systemd`{.inlineCode} commands:

``` {.programlisting .snippet-con}
$ systemctl --user start 'podman-compose@golang-redis
$ systemctl --user stop 'podman-compose@golang-redis
$ systemctl --user status 'podman-compose@golang-redis
```

The[]{#Chapter_13.xhtml#idx_28f31242 .index-entry
index-entry="podman-compose project"} landscape of multi-container
orchestration on a single host has matured
significantly.[]{.sentence-end} While the
`podman`{.inlineCode}`-compose`{.inlineCode} project has made great
steps forward in compatibility and offers unique features, such as the
ability to automatically group services into Pods, this *pod-first*
approach can sometimes introduce unexpected friction with complex
Compose files designed for Docker\'s individual-container model.

For most developers, the `docker`{.inlineCode}`-compose`{.inlineCode}
command remains the gold standard for high-fidelity
compatibility.[]{.sentence-end} It allows you to run existing stacks
without the *debugging tax* often associated with translation layers.

While `podman`{.inlineCode}`-compose`{.inlineCode} continues to evolve
as a community-driven, Python-based alternative, Podman itself is moving
toward Quadlets, which we will explore in the next chapter, as the
preferred *native* method for managing service lifecycles via
`systemd`{.inlineCode} in a single system and through
`podman`{.inlineCode}` `{.inlineCode}`kube`{.inlineCode} moving to
production multi-node environments such as OpenShift or Kubernetes.

# Summary {#Chapter_13.xhtml#h1_327 .heading-1}

In this chapter, we learned how to manage a full migration from Docker
to Podman.

We covered how to migrate images and create command aliases, and we
inspected the command compatibility matrix.[]{.sentence-end} Here, we
provided a detailed overview of the different behaviors of specific
commands and the different commands that are implemented in the two
container engines---that is, Docker and Podman.

Then, we learned how to migrate Docker Compose by illustrating native
Podman support for the `docker`{.inlineCode}`-compose`{.inlineCode}
command and the `podman`{.inlineCode}`-compose`{.inlineCode} alternative
utility.

In the next chapter of this book, we will learn how to interact with
`s`{.inlineCode}`ystemd`{.inlineCode} by generating custom service units
and turning containers into services that are started automatically
inside the host.[]{.sentence-end} Then, we\'ll look at
Kubernetes-oriented orchestration, where we will learn how to generate
Kubernetes resources from running containers and pods and run them in
Podman or Kubernetes natively.

# Further reading {#Chapter_13.xhtml#h1_328 .heading-1}

To learn more about the topics that were covered in this chapter, take a
look at the following resources:

-   Docker Awesome Compose:
    [[https://github.com/docker/awesome-compose]{.url}](https://github.com/docker/awesome-compose){style="text-decoration: none;"}
-   Podman Compose project on GitHub:
    [[https://github.com/containers/podman-compose]{.url}](https://github.com/containers/podman-compose){style="text-decoration: none;"}
-   Red Hat blog introduction to Docker Compose support in Podman:
    [[https://www.redhat.com/sysadmin/podman-docker-compose]{.url}](https://www.redhat.com/sysadmin/podman-docker-compose){style="text-decoration: none;"}
-   Twelve-Factor App:
    [[https://12factor.net/]{.url}](https://12factor.net/){style="text-decoration: none;"}
-   Podman `man`{.inlineCode} page:

# Get This Book's PDF Version and Exclusive Extras {#Chapter_13.xhtml#h1_329 .heading-1}

Scan the QR code (or go to
[[packtpub.com/unlock]{.url}](https://packtpub.com/unlock){style="text-decoration: none;"}).[]{.sentence-end}
Search for this book by name, confirm the edition, and then follow the
steps on the page.

![Image](images/B31467_13_3.png){style="width:25%;"}

![Image](images/B31467_13_4.png){style="width:25%;"}

*Note: Keep your invoice handy.[]{.sentence-end} Purchases made directly
from* *Packt* *don't require one.*


[]{#Chapter_14.xhtml}

 {.section .chapter-first}