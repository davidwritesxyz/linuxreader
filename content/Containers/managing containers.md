# 4 {#Chapter_4.xhtml#h1_114 .chapterNumber}

# Managing Running Containers {#Chapter_4.xhtml#h1_115 .chapterTitle}

In the previous chapter, we learned how to set up the environment to run
containers with Podman, covering binary installation for the major
distributions, system configuration files, and a first example container
run to verify that our setup was correct.[]{.sentence-end} This chapter
will offer a more detailed overview of container execution, how to
manage and inspect running containers, and how to group containers in
pods.[]{.sentence-end} This chapter is important for gaining the right
knowledge and expertise to start our experience as a system
administrator for container technologies.

In this chapter, we\'re going to cover the following main topics:

-   Managing container images
-   Operations with running containers
-   Inspecting container information
-   Capturing logs from containers
-   Executing processes into a running container
-   Running containers in pods

# Technical requirements {#Chapter_4.xhtml#h1_116 .heading-1}

Before proceeding with this chapter and its exercises, a machine with a
working Podman instance is required.[]{.sentence-end} As stated in
*[Chapter 3](#Chapter_3.xhtml#h1_83){.chapref}*, *Running* *the First
Container*, all the examples in the book are executed on a Fedora 40
system, but can be reproduced on an OS of your choice.

Finally, a good understanding of the topics covered in the previous
chapters is useful to easily grasp concepts regarding OCI images and
container execution.

# Managing container images {#Chapter_4.xhtml#h1_117 .heading-1}

In this section, we will see how to find and pull (download) an image in
the local system, as well as how to inspect its
contents.[]{.sentence-end} When a container is created and run for the
first time, Podman takes care of pulling the related image
automatically.[]{.sentence-end} However, being able to pull and inspect
images in advance gives []{#Chapter_4.xhtml#idx_6271db0c .index-entry
index-entry="container images:managing"}some valuable advantages, the
first being that a container executes faster when images are already
available in the machine\'s local store.

As we stated in the previous chapters, containers are a way to isolate
processes in a sandboxed environment with separate namespaces and
resource allocation.

The filesystem mounted in the container is provided by the OCI image
described in *[Chapter 2](#Chapter_2.xhtml#h1_52){.chapref}*, *Comparing
Podman and Docker*.

OCI images are stored and distributed by specialized
[]{#Chapter_4.xhtml#idx_c8db77f3 .index-entry
index-entry="container registry"}services called container
registries.[]{.sentence-end} A container registry stores images and
metadata and exposes simple REST API services to enable users to push
and pull images.

There are essentially two types of registries: public and
[]{#Chapter_4.xhtml#idx_accae7ba .index-entry
index-entry="public registry"}private.[]{.sentence-end} A **public
registry** is accessible as a public service (with or without
authentication).[]{.sentence-end} The main public registries, such as
`docker.io`{.inlineCode}, `gcr.io`{.inlineCode}, or
`quay.io`{.inlineCode}, are also used as the image repositories of
larger open source projects.[]{.sentence-end} Most of these registries
will also offer private hosting for a fee.

**Private registries** are[]{#Chapter_4.xhtml#idx_18f3de9a .index-entry
index-entry="private registries"} deployed and managed inside an
organization and can be more focused on security and content
filtering.[]{.sentence-end} The main container registry projects
nowadays are graduated under the CNCF
([[https://landscape.cncf.io/card-mode?category=container-registry&grouping=category]{.url}](https://landscape.cncf.io/card-mode?category=container-registry&grouping=category){style="text-decoration: none;"})
and offer advanced enterprise features to manage multitenancy,
authentication, and **role-based access control** (**RBAC**), as well as
image vulnerability scanning and image signing.

In *[Chapter 9](#Chapter_9.xhtml#h1_219){.chapref}*, *Pushing Images to
a Container Registry*, we will provide more details and examples of
interaction with container registries.

The largest part of public and private registries exposes Docker
Registry HTTP API V2
([[https://docs.docker.com/registry/spec/api/]{.url}](https://docs.docker.com/registry/spec/api/){style="text-decoration: none;"}).[]{.sentence-end}
This being an HTTP-based REST API, users can interact with the
[]{#Chapter_4.xhtml#idx_7df9c3b1 .index-entry
index-entry="container images:managing"}registry with a simple
`curl`{.inlineCode} command or design their own custom clients.

Podman offers a CLI to interact with public and private container
registries, manage logins when registry authentication is required,
search for image repositories by passing a string pattern, and handle
locally cached images.

## Searching for images {#Chapter_4.xhtml#h2_118 .heading-2}

The first command we will learn to use[]{#Chapter_4.xhtml#idx_73692bf5
.index-entry index-entry="container images:searching for"} to search
images across multiple registries is the `podman search`{.inlineCode}
command.[]{.sentence-end} The following example shows how to search an
nginx image:

``` {.programlisting .snippet-con}
# podman search nginx
```

The preceding command will produce an output with many entries from all
the whitelisted registries (see the *Preparing your environment*
\|*Customizing container registries\'* *search lists* section of
*[Chapter 3](#Chapter_3.xhtml#h1_83){.chapref}*, *Running the First
Container*).[]{.sentence-end} The output will be a little clumsy, with
many entries from unknown and unreliable repositories.[]{.sentence-end}
It\'s also worth noting that most public registries have web-based
search that is of significantly higher quality than the results
available through `podman search`{.inlineCode}.

In general, the `podman search`{.inlineCode} command accepts the
following pattern:

``` {.programlisting .snippet-con}
podman search [options] TERM
```

Here, `TERM`{.inlineCode} is the search argument.[]{.sentence-end} The
resulting output of a search has the following fields:

-   `INDEX`{.inlineCode}: The registry indexing the image
-   `NAME`{.inlineCode}: The full name of the image, including registry
    name and associated namespaces
-   `DESCRIPTION`{.inlineCode}: A short description of the image role
-   `STARS`{.inlineCode}: The number of stars given by users (available
    only on registries supporting this feature, such as
    `docker.io`{.inlineCode})
-   `OFFICIAL`{.inlineCode}: A Boolean
    to[]{#Chapter_4.xhtml#idx_14fe50a1 .index-entry
    index-entry="container images:searching for"} specify if the image
    is official
-   `AUTOMATED`{.inlineCode}: A field set to `OK`{.inlineCode} if the
    image is automated

 packt_tip
**Important** **note**

Never trust unknown repositories and always prefer official
images.[]{.sentence-end} When pulling images from a niche project, try
to understand the content of the image before running
it.[]{.sentence-end} Remember that an attacker could hide malicious code
that could be executed inside containers.

Even trusted repositories can be compromised in some
cases.[]{.sentence-end} In enterprise scenarios, implement image
signature verification to avoid image tampering.


It is possible to apply filters to the search and refine the
output.[]{.sentence-end} For example, to refine the search and print
only official images, we can add the following filtering option, which
only prints out images with the `is-official`{.inlineCode} flag:

``` {.programlisting .snippet-con}
# podman search nginx --filter=is-official
```

This command will print one line pointing to
`docker.io/library/nginx:latest`{.inlineCode}.[]{.sentence-end} This
official image is maintained by the nginx community and can be used more
confidently.

Users can refine the output format of the command.[]{.sentence-end} The
following example shows how to print only the image registry and the
image name:

``` {.programlisting .snippet-con}
# podman search fedora  \
  --filter is-official \
  --format "table {{.Index}} {{.Name}}"

INDEX       NAME
docker.io   docker.io/library/fedora
```

The output image name has []{#Chapter_4.xhtml#idx_64f78c83 .index-entry
index-entry="container images:searching for"}a standard naming pattern
that deserves a detailed description.[]{.sentence-end} The standard
format is shown here:

``` {.programlisting .snippet-con}
<registry>[:<port>]/[<namespace>/]<name>:<tag>
```

Let\'s describe the preceding fields in detail, as follows:

-   `registry`{.inlineCode}: This contains the registry the image is
    stored in.[]{.sentence-end} The nginx image in our example is stored
    in the `docker.io`{.inlineCode} public registry.[]{.sentence-end}
    Optionally, it is possible to specify a custom
    []{#Chapter_4.xhtml#idx_b4efced1 .index-entry
    index-entry="Transmission Control Protocol (TCP)"}port number for
    the registry.[]{.sentence-end} By default, registries expose the
    `5000`{.inlineCode} **Transmission Control Protocol** (**TCP**)
    port.
-   `namespace`{.inlineCode}: This field could be optional and provides
    a hierarchy structure that is useful for distinguishing the image
    context from the provider.[]{.sentence-end} The namespace could
    represent the parent organization, the username of the owner of the
    repository, or the image role.
-   `name`{.inlineCode}: This contains the name of the private/public
    image repository where all the tags are stored.[]{.sentence-end} It
    is often referred to as the application name (that is, nginx).
-   `tag`{.inlineCode}: Every image stored in the
    []{#Chapter_4.xhtml#idx_e4baea6b .index-entry
    index-entry="Secure Hash Algorithm 256 (SHA256)"}registry has a
    unique tag, mapped to a **Secure Hash Algorithm 256** (**SHA256**)
    digest.[]{.sentence-end} The generic `:latest`{.inlineCode} tag can
    be omitted in the image name.

The generic search hides the image tags by default.[]{.sentence-end} To
show all available tags for a given repository, we can use the
`-`{.inlineCode}`list-tags`{.inlineCode} option for a given image name,
as follows:

``` {.programlisting .snippet-con}
# podman search quay.io/prometheus/prometheus --list-tags
NAME                           TAG
quay.io/prometheus/prometheus  0.19.0
quay.io/prometheus/prometheus  0.19.1
quay.io/prometheus/prometheus  0.19.2
quay.io/prometheus/prometheus  0.19.3
quay.io/prometheus/prometheus  0.20.0
quay.io/prometheus/prometheus  latest
quay.io/prometheus/prometheus  main
quay.io/prometheus/prometheus  master
quay.io/prometheus/prometheus  v1.0.0
  
[...output omitted...]
```

This option is really useful for[]{#Chapter_4.xhtml#idx_80ca97fd
.index-entry index-entry="container images:searching for"} finding a
specific image tag in the registry, often associated with a release
version of the application/runtime.

 packt_tip
**Important** **note**

Using the `:latest`{.inlineCode} tag can lead to image versioning issues
since it is not a descriptive tag.[]{.sentence-end} Also, it is usually
expected to point to the latest image version.[]{.sentence-end}
Unfortunately, this is not always true since an untagged image could
retain the latest tag while the latest pushed image could have a
different tag.[]{.sentence-end} It is up to the repository maintainer to
apply tags correctly.[]{.sentence-end} If the repository uses semantic
versioning, the best option is to pull the most recent version
tag.[]{.sentence-end} In addition to that, another risk could be the
maintainers pushing incompatible updates to \``:latest`{.inlineCode}\`
and breaking any existing deployment.


## Pulling and viewing images {#Chapter_4.xhtml#h2_119 .heading-2}

Once we have []{#Chapter_4.xhtml#idx_97852fc2 .index-entry
index-entry="container images:pulling"}found our desired image, it can
be downloaded using the `podman pull`{.inlineCode} command, as follows:

``` {.programlisting .snippet-con}
# podman pull docker.io/library/nginx:latest
```

Notice the root user for running the[]{#Chapter_4.xhtml#idx_bac0f42f
.index-entry index-entry="container images:viewing"} Podman
command.[]{.sentence-end} In this case, we are pulling the image as
root, and its layers and metadata are stored in the
`/var/lib/containers/storage`{.inlineCode} path.

We can run the same command as a standard user by executing the command
in a standard user\'s shell, like this:

``` {.programlisting .snippet-con}
$ podman pull docker.io/library/nginx:latest
```

In this case, the image will be downloaded in the user\'s home directory
under `$HOME/.local/share/containers/storage/`{.inlineCode} and will be
available to run rootless containers for the user who executed the
command.

 packt_tip
**Important** **note**

You could also try pulling container images using
aliases.[]{.sentence-end} For example, the
\``podman pull nginx`{.inlineCode}\` command without specifying the
registry could try to resolve some of the images, and in case there are
some doubts, a prompt pops up asking the user what registry to pull
from.


Users can inspect all locally []{#Chapter_4.xhtml#idx_f955e564
.index-entry index-entry="container images:viewing"}cached images with
the `podman images`{.inlineCode} command,
as[]{#Chapter_4.xhtml#idx_8eab3553 .index-entry
index-entry="container images:pulling"} illustrated here:

``` {.programlisting .snippet-con}
# podman images
REPOSITORY               TAG         IMAGE ID      CREATED     SIZE
docker.io/library/nginx  latest      7f553e8bbc89  6 days ago  196 MB

[...omitted output...]
```

The output shows the image repository name, its tag, the image
**identifier** (**ID**), the creation date, and the image
size.[]{.sentence-end} It is very useful to keep an updated view of the
images available in the local store and understand which ones are
obsolete.

The `podman images`{.inlineCode} command also supports many options (a
complete list is available by executing the
`man podman-images`{.inlineCode} command).[]{.sentence-end} One of the
more interesting options is `-`{.inlineCode}`sort`{.inlineCode}, which
can be used to sort images by size, date, ID, repository, or
tag.[]{.sentence-end} For example, we could print images sorted by
creation date to find out the most obsolete ones, as follows:

``` {.programlisting .snippet-con}
# podman images --sort=created
```

Another couple of very useful options are the
`-`{.inlineCode}`all`{.inlineCode} (or `-`{.inlineCode}`a`{.inlineCode})
and `-`{.inlineCode}`quiet`{.inlineCode} (or
`-`{.inlineCode}`q`{.inlineCode}) options.[]{.sentence-end} Together,
they can be []{#Chapter_4.xhtml#idx_59f9eadd .index-entry
index-entry="container images:pulling"}combined to print only the image
IDs of all the locally stored images, even intermediate image layers.

 note
**Important** **note**

In the context of **Open Container Initiative** (**OCI**) images,
intermediate[]{#Chapter_4.xhtml#idx_03ab4d0a .index-entry
index-entry="Open Container Initiative (OCI)"} layers are the
individual, read-only filesystem snapshots created during the build
process.

When you build an image (using a Dockerfile or Containerfile), each
instruction, such as `RUN`{.inlineCode}, `COPY`{.inlineCode}, or
`ADD`{.inlineCode}, creates a new layer.[]{.sentence-end} These layers
are stacked on top of one another to form the final image.


The command will print output[]{#Chapter_4.xhtml#idx_8e235787
.index-entry index-entry="container images:viewing"} similar to the
following example:

``` {.programlisting .snippet-con}
# podman images -qa
ad4c705f24d3
a56f85702a94
b5c5125e3fee
4d7fc5917f3e
625707533167
f881f1aa4d65
96ab2a326180
```

Listing and showing the images already pulled on a system is not the
most interesting part of the job! Let\'s discover how to inspect images
with their configuration and contents in the next section.

## Inspecting images\' configurations and contents {#Chapter_4.xhtml#h2_120 .heading-2}

To inspect the configuration of []{#Chapter_4.xhtml#idx_28d6fcfd
.index-entry
index-entry="container images:configurations and contents, inspecting"}a
pulled image, the `podman image inspect`{.inlineCode} (or the shorter
`podman inspect`{.inlineCode}) command comes to help us, as illustrated
here:

``` {.programlisting .snippet-con}
# podman inspect docker.io/library/nginx:latest
```

The printed output will be a JSON-formatted object containing the image
config, architecture, layers, labels, annotation, and the image build
history.

The image history shows the creation history of every layer and is very
useful for understanding how the image was built when the Dockerfile or
the Containerfile is not available.

Since the output is a JSON object, we can extract single fields to
collect specific data or use them as input parameters for other
commands.

The following example prints out the command executed when a container
is created upon this image:

``` {.programlisting .snippet-con}
# podman inspect docker.io/library/nginx:latest \
--format "{{ .Config.Cmd }}"
[nginx -g daemon off;]
```

Notice that the formatted output is managed as a Go template.

Sometimes, especially when troubleshooting container issues, the
inspection of an image must go further than a simple
[]{#Chapter_4.xhtml#idx_3788cda4 .index-entry
index-entry="container images:configurations and contents, inspecting"}configuration
check.[]{.sentence-end} On occasions, we need to inspect the filesystem
content of an image.[]{.sentence-end} To achieve this result, Podman
offers the useful `podman image mount`{.inlineCode} command.

The following example mounts the image and prints its mount path:

``` {.programlisting .snippet-con}
# podman image mount docker.io/library/nginx
/var/lib/containers/storage/overlay/11174639fa910c6d29b6f169964a8388815d85b7908b8825946e276e7242aea5/merged
```

If we run a simple `ls`{.inlineCode} command in the provided path, we
will see the image filesystem, composed from its various merged layers,
as follows:

``` {.programlisting .snippet-con}
# ls -al /var/lib/containers/storage/overlay/11174639fa910c6d29b6f169964a8388815d85b7908b8825946e276e7242aea5/merged/
total 20
dr-xr-xr-x. 1 root root   38 Oct  9 10:04 .
drwx------. 1 root root   46 Oct  9 10:04 ..
lrwxrwxrwx. 1 root root    7 Sep 26 00:00 bin -> usr/bin
drwxr-xr-x. 1 root root    0 Aug 14 16:10 boot
drwxr-xr-x. 1 root root    0 Sep 26 00:00 dev
drwxr-xr-x. 1 root root   54 Oct  3 00:58 docker-entrypoint.d
-rwxr-xr-x. 1 root root 1620 Oct  3 00:57 docker-entrypoint.sh
drwxr-xr-x. 1 root root  366 Oct  3 00:58 etc
drwxr-xr-x. 1 root root    0 Aug 14 16:10 home
lrwxrwxrwx. 1 root root    7 Sep 26 00:00 lib -> usr/lib
lrwxrwxrwx. 1 root root    9 Sep 26 00:00 lib64 -> usr/lib64
drwxr-xr-x. 1 root root    0 Sep 26 00:00 media
drwxr-xr-x. 1 root root    0 Sep 26 00:00 mnt
drwxr-xr-x. 1 root root    0 Sep 26 00:00 opt
drwxr-xr-x. 1 root root    0 Aug 14 16:10 proc
drwx------. 1 root root   30 Sep 26 00:00 root
drwxr-xr-x. 1 root root    8 Sep 26 00:00 run
lrwxrwxrwx. 1 root root    8 Sep 26 00:00 sbin -> usr/sbin
drwxr-xr-x. 1 root root    0 Sep 26 00:00 srv
drwxr-xr-x. 1 root root    0 Aug 14 16:10 sys
drwxrwxrwt. 1 root root    0 Sep 26 00:00 tmp
drwxr-xr-x. 1 root root   40 Sep 26 00:00 usr
drwxr-xr-x. 1 root root   22 Sep 26 00:00 var
```

To unmount the image, simply run the
`podman image`{.inlineCode}` unmount`{.inlineCode} command, as follows:

``` {.programlisting .snippet-con}
# podman image unmount docker.io/library/nginx
```

Mounting images in rootless mode is a bit different since this execution
mode only supports manual mounting of the **Virtual File System**
(**VFS**) storage driver.[]{.sentence-end} Since
[]{#Chapter_4.xhtml#idx_664aa361 .index-entry
index-entry="Virtual File System (VFS)"}we are working with a default
OverlayFS storage driver, the
`mount`{.inlineCode}/`unmount`{.inlineCode} commands would not
work.[]{.sentence-end} The requirement is to run the
`podman unshare`{.inlineCode} command first.[]{.sentence-end} It
executes a new shell process inside a new
[]{#Chapter_4.xhtml#idx_629ea7e8 .index-entry
index-entry="user ID (UID)"}namespace where the current **user ID**
(**UID**) and **globally unique ID** (**GID**) are mapped
to[]{#Chapter_4.xhtml#idx_670b0854 .index-entry
index-entry="globally unique ID (GID)"} UID 0 and GID 0,
respectively.[]{.sentence-end} From now on, we have elevated privileges
to run the `podman mount`{.inlineCode} command.[]{.sentence-end} Let\'s
see an example here:

``` {.programlisting .snippet-con}
$ podman unshare
# podman image mount docker.io/library/nginx:latest
.local/share/containers/storage/overlay/11174639fa910c6d29b6f169964a8388815d85b7908b8825946e276e7242aea5/merged
```

Notice the mount point is now in the
`<`{.inlineCode}`username`{.inlineCode}`>`{.inlineCode} home directory.

To unmount, simply run the `podman `{.inlineCode}`unmount`{.inlineCode}
command, as follows:

``` {.programlisting .snippet-con}
# podman image unmount docker.io/library/nginx:latest
7f553e8bbc897571642d836b31eaf6ecbe395d7641c2b24291356ed28f3f2bd0
# exit
```

The `exit`{.inlineCode} command is necessary to
[]{#Chapter_4.xhtml#idx_a5d584bf .index-entry
index-entry="container images:configurations and contents, inspecting"}exit
the temporary unshared namespace.

## Deleting images {#Chapter_4.xhtml#h2_121 .heading-2}

To delete a local store image, we can use the `podman rmi`{.inlineCode}
command.[]{.sentence-end} The following example deletes the nginx
image[]{#Chapter_4.xhtml#idx_a6c74e9a .index-entry
index-entry="container images:deleting"} pulled before:

``` {.programlisting .snippet-con}
# podman rmi docker.io/library/nginx:latest
Untagged: docker.io/library/nginx:latest
Deleted: 7f553e8bbc897571642d836b31eaf6ecbe395d7641c2b24291356ed28f3f2bd0
```

The same command works in rootless mode when executed by a standard user
against its home local store.

To remove all the cached images, use the following example, which relies
on shell command expansion to get a full list of image IDs:

``` {.programlisting .snippet-con}
# podman rmi $(podman images -qa)
```

Notice the sharp (`#`{.inlineCode}) symbol at the beginning of the line,
which tells us that the command is executed as root.

The next command removes all images in a regular user\'s local cache
(notice the dollar symbol at the beginning of the line):

``` {.programlisting .snippet-con}
$ podman rmi $(podman images -qa)
```

 packt_tip
**Important** **note**

The `podman rmi`{.inlineCode} command fails to remove images that are
currently in use from a container.[]{.sentence-end} First, stop and
delete the containers using the []{#Chapter_4.xhtml#idx_82491237
.index-entry index-entry="podman rmi command"}blocked images, and then
run the command again.[]{.sentence-end} Alternatively, we can use the
`--force`{.inlineCode} option, which will remove images that are in use,
but it will also remove all containers that are using the image.


Podman also offers a simpler []{#Chapter_4.xhtml#idx_75e2e8fa
.index-entry index-entry="container images:deleting"}way to clean up
dangling or unused images---the `podman image prune`{.inlineCode}
command.[]{.sentence-end} It deletes every image that is not in use,
leaving only those associated with containers, running or stopped.

The following example deletes all unused images without asking for
confirmation:

``` {.programlisting .snippet-con}
$ sudo podman image prune -af
```

The same command applies in rootless mode, deleting only images in the
user\'s home local store.[]{.sentence-end} With this, we have learned
how to manage container images on our machine.[]{.sentence-end} Let\'s
now learn how to handle and check running containers.

# Operations with running containers {#Chapter_4.xhtml#h1_122 .heading-1}

In *[Chapter 2](#Chapter_2.xhtml#h1_52){.chapref}*, *Comparing Podman
and Docker*, we learned in the *Running your first container* section
how to run a container with basic examples, involving the execution of a
Bash process inside a Fedora container[]{#Chapter_4.xhtml#idx_0cb0a925
.index-entry index-entry="running containers:operations, performing"}
and an `httpd`{.inlineCode} server, which was also helpful for learning
how to expose containers externally.

We will now explore a set of commands used to monitor and check our
running containers and gain insights into their behavior.

## Viewing and handling container status {#Chapter_4.xhtml#h2_123 .heading-2}

Let\'s start by running a []{#Chapter_4.xhtml#idx_2b6156d8 .index-entry
index-entry="running containers:status, viewing"}simple container and
exposing it on port `8080`{.inlineCode} to make it
[]{#Chapter_4.xhtml#idx_a3e86649 .index-entry
index-entry="running containers:status, handling"}accessible externally,
as follows:

``` {.programlisting .snippet-con}
$ podman run -d -p 8080:80 docker.io/library/nginx
```

The preceding example is run in rootless mode, but the same can be
applied as a root user by prepending `sudo`{.inlineCode} to the
[]{#Chapter_4.xhtml#idx_ad9d7f14 .index-entry
index-entry="running containers:status, viewing"}command.[]{.sentence-end}
In this case, it was simply not necessary to have a container executed
in that way.

 note
**Important** **note**

Rootless containers give an extra[]{#Chapter_4.xhtml#idx_409bd810
.index-entry index-entry="rootless containers"} security
advantage.[]{.sentence-end} If a malicious process breaks the container
isolation, maybe leveraging a vulnerability on the host, it will at best
gain the privileges of the user who started the rootless container.


Now that our container is up and running and
[]{#Chapter_4.xhtml#idx_b4eafd79 .index-entry
index-entry="running containers:status, handling"}ready to serve, we can
test it by running a `curl`{.inlineCode} command on the localhost, which
should produce an HTML default output like this:

``` {.programlisting .snippet-con}
$ curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
```

Obviously, an empty nginx server without contents to serve is useless,
but we will learn how to serve custom contents by using volumes or
[]{#Chapter_4.xhtml#idx_c854c8d0 .index-entry
index-entry="running containers:status, handling"}building custom images
later, in the following chapters.

The first command we can use to[]{#Chapter_4.xhtml#idx_c9a30795
.index-entry index-entry="running containers:status, viewing"} check our
container is `podman ps`{.inlineCode}.[]{.sentence-end} This simply
prints out useful information from the running containers, with the
option of customizing and sorting the output.[]{.sentence-end} Let\'s
run the command in our host and see what is printed, as follows:

``` {.programlisting .snippet-con}
$ podman ps
CONTAINER ID  IMAGE                           COMMAND               CREATED         STATUS         PORTS                         NAMES
d19dbb81a0f3  docker.io/library/nginx:latest  nginx -g daemon o...  34 seconds ago  Up 34 seconds  0.0.0.0:8080->80/tcp, 80/tcp  admiring_bouman
```

The output produces some interesting information about running
containers, as detailed here:

-   `CONTAINER ID`{.inlineCode}: Every new container gets a unique
    hexadecimal ID.[]{.sentence-end} The full ID has a length of 64
    characters, and a shortened portion of 12 characters is printed in
    the `podman ps`{.inlineCode} output.
-   `IMAGE`{.inlineCode}: The image used by the container.
-   `COMMAND`{.inlineCode}: The command executed inside the container.
-   `CREATED`{.inlineCode}: The creation date of the container.
-   `STATUS`{.inlineCode}: The current container status.
-   `PORTS`{.inlineCode}: The network ports opened in the
    container.[]{.sentence-end} When a port mapping is applied, we can
    see one or more host `ip`{.inlineCode}`:port`{.inlineCode} pairs
    mapped to the container ports with an arrow sign.[]{.sentence-end}
    For example, the
    `0.0.0.0:8080-`{.inlineCode}`>`{.inlineCode}`80/tcp`{.inlineCode}
    string means that the `8080/tcp`{.inlineCode} host port is exposed
    on all the listening interfaces and is mapped to the
    `80/tcp`{.inlineCode} container port.
-   `NAMES`{.inlineCode}: The container name.[]{.sentence-end} This can
    be assigned by the user or be randomly generated by the container
    engine.

 note
**Tip**

Notice the randomly generated name in the last column of the
output.[]{.sentence-end} Podman continues the Docker *tradition* of
generating random names using adjectives in the left part of the name
and notable scientists and hackers in the right part.[]{.sentence-end}
Indeed, Podman still uses the same
`github.com/docker/docker/namesgenerator`{.inlineCode} Docker package,
included in the vendor directory of the project.


To get a full list of both running and stopped
[]{#Chapter_4.xhtml#idx_97ecf975 .index-entry
index-entry="running containers:status, handling"}containers, we can add
an `-`{.inlineCode}`a`{.inlineCode} option to the
command.[]{.sentence-end} To demonstrate this, we
[]{#Chapter_4.xhtml#idx_5d8330db .index-entry
index-entry="running containers:status, viewing"}first introduce the
`podman stop`{.inlineCode} command.[]{.sentence-end} This changes the
container status to stopped and sends a `SIGTERM`{.inlineCode} signal to
the processes running inside the container.[]{.sentence-end} If the
container becomes unresponsive, it sends a `SIGKILL`{.inlineCode} signal
after a given timeout of 10 seconds.

Let\'s try to stop the previous container and check its state by
executing the following code:

``` {.programlisting .snippet-con}
$ podman stop d19dbb81a0f3
$ podman ps
```

This time, `podman ps`{.inlineCode} produced an empty
output.[]{.sentence-end} This is because the container state is
stopped.[]{.sentence-end} To get a full list of both running and stopped
containers, run the following command:

``` {.programlisting .snippet-con}
$ podman ps -a
CONTAINER ID  IMAGE                           COMMAND               CREATED             STATUS                     PORTS                         NAMES
d19dbb81a0f3  docker.io/library/nginx:latest  nginx -g daemon o...  About a minute ago  Exited (0) 20 seconds ago  0.0.0.0:8080->80/tcp, 80/tcp  admiring_bouman
```

Notice the status of the container, which states that the container has
exited with a `0`{.inlineCode} exit code.

The stopped container can be resumed by running the
`podman start`{.inlineCode} command, as follows:

``` {.programlisting .snippet-con}
$ podman start d19dbb81a0f3
```

This command simply starts the container we stopped before again.

If we now check the container status again, we will see it is up and
running, as indicated here:

``` {.programlisting .snippet-con}
$ podman ps
CONTAINER ID  IMAGE                           COMMAND               CREATED             STATUS         PORTS                         NAMES
d19dbb81a0f3  docker.io/library/nginx:latest  nginx -g daemon o...  About a minute ago  Up 13 seconds  0.0.0.0:8080->80/tcp, 80/tcp  admiring_bouman
```

Podman keeps the container configuration, storage, and
[]{#Chapter_4.xhtml#idx_b8cd2e98 .index-entry
index-entry="running containers:status, handling"}metadata as long as it
is in a stopped state.[]{.sentence-end} Anyway, when we
resume[]{#Chapter_4.xhtml#idx_95b62e39 .index-entry
index-entry="running containers:status, viewing"} the container, we
start a new process inside it.

For more options, see[]{#Chapter_4.xhtml#idx_9bd7e1a1 .index-entry
index-entry="manual (man) page"} the related **manual** (**man**) page
(`man podman-start`{.inlineCode}).

If we simply need to restart a running container, we can use the
`podman restart`{.inlineCode} command, as follows:

``` {.programlisting .snippet-con}
$ podman restart <Container_ID_or_Name>
```

This command has the effect of immediately restarting the container,
using the same data and process as the old one.[]{.sentence-end} While
we reuse the filesystem and configuration, all namespaces will be
different, which can fix many networking issues, as all configurations
will be performed again.[]{.sentence-end} The
`podman start`{.inlineCode} command can also be used to start containers
that have been previously created but not run.[]{.sentence-end} To
create a container without starting it, use the
`podman create`{.inlineCode} command.[]{.sentence-end} The following
example creates a container but does not start it:

``` {.programlisting .snippet-con}
$ podman create -p 8080:80 docker.io/library/nginx
```

To start it, run `podman start`{.inlineCode} on the created container ID
or name, as follows:

``` {.programlisting .snippet-con}
$ podman start <Container_ID_or_Name>
```

This command is very useful for preparing an environment without running
it or for mounting a container filesystem for a regular user (non-root),
as in the following example:

``` {.programlisting .snippet-con}
$ podman unshare
$ podman container mount <Container_ID_or_Name>
/home/<username>/.local/share/containers/storage/overlay/bf9d8df299436d80dece200a23e1b8b957f987a254a656ef94cdc56669823b5c/merged
```

 note
**Good to** **know**

The `podman container mount`{.inlineCode} command (or the shorter
`podman mount`{.inlineCode}) is used to mount a container\'s root
filesystem to a location []{#Chapter_4.xhtml#idx_516ab8fb .index-entry
index-entry="podman container mount command"}on your host
machine.[]{.sentence-end} This allows you to browse, read, and modify
the files inside a container directly from the host\'s terminal or GUI,
without needing to execute a shell inside the container itself.


Let\'s now introduce a very frequently used command:
`podman rm`{.inlineCode}.[]{.sentence-end} As the name indicates, it is
used to remove containers from the[]{#Chapter_4.xhtml#idx_2d11b376
.index-entry index-entry="running containers:status, handling"}
host.[]{.sentence-end} By default, it removes stopped containers, but it
can be forced to remove running[]{#Chapter_4.xhtml#idx_0f77bd93
.index-entry index-entry="running containers:status, viewing"}
containers with the `-`{.inlineCode}`f`{.inlineCode} option.

Using the container from the previous example, if we stop it again and
issue the `podman rm`{.inlineCode} command, as illustrated in the
following commands, all the container storage, configs, and metadata
will be discarded:

``` {.programlisting .snippet-con}
$ podman stop d19dbb81a0f3
$ podman rm d19dbb81a0f3
```

If we now run a `podman ps`{.inlineCode} command again, even with the
`-`{.inlineCode}`a`{.inlineCode} option, we will get an empty list, as
illustrated here:

``` {.programlisting .snippet-con}
$ podman ps -a
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
```

For more details, please inspect the command man page
(`man podman-rm`{.inlineCode}).

Sometimes, it is useful---just as with images---to print only the
container ID with the `-`{.inlineCode}`q`{.inlineCode}
option.[]{.sentence-end} And when combined with the
`-`{.inlineCode}`a`{.inlineCode} option, this
[]{#Chapter_4.xhtml#idx_3dea2cb7 .index-entry
index-entry="running containers:status, handling"}can print a list of
all stopped and running containers on the host.[]{.sentence-end} Let\'s
try another example here:

``` {.programlisting .snippet-con}
$ for i in {1..5}; do podman run -d docker.io/library/nginx; done
```

It\'s interesting to notice that we[]{#Chapter_4.xhtml#idx_90accbfb
.index-entry index-entry="running containers:status, viewing"} have used
a shell loop to start five identical containers, this time without any
port mapping---just plain nginx containers.[]{.sentence-end} We can
inspect their IDs with the following command:

``` {.programlisting .snippet-con}
$ podman ps -qa
b38ebfed5921
6204efc6d6b2
762967d87657
269f1affb699
1161072ec559
```

How can we stop and remove all our running containers quickly? We can
use **shell expansion** to combine it with other commands and reach the
desired result.[]{.sentence-end} Shell expansion is a powerful tool that
runs the command inside round parentheses and lets us pass the output
string as arguments to the external command, as illustrated in the
following commands:

``` {.programlisting .snippet-con}
$ podman stop $(podman ps -qa)
$ podman rm $(podman ps -qa)
```

The two commands stopped all the running containers, identified by their
IDs, and removed them from the host.

Alternatively, we can use an alias for all containers,
`-a`{.inlineCode}, which most commands, such as
`podman start`{.inlineCode}, `podman stop`{.inlineCode}, and
`podman rm`{.inlineCode} support.

The `podman ps`{.inlineCode} command enables users to refine their
output by applying specific filters.[]{.sentence-end} A full list of all
applicable filters is available on the `podman-ps`{.inlineCode} man
page.[]{.sentence-end} A simple but useful application is the status
filter, which enables users to print only containers in a specific
condition.[]{.sentence-end} Possible statuses are
`created`{.inlineCode}, `exited`{.inlineCode}, `paused`{.inlineCode},
`running`{.inlineCode}, and `unknown`{.inlineCode}.

The following example only prints containers in an `exited`{.inlineCode}
status:

``` {.programlisting .snippet-con}
$ podman ps --filter status=exited
```

Again, we can leverage the power of shell expansion to remove nothing
but the exited containers, as follows:

``` {.programlisting .snippet-con}
$ podman rm $(podman ps -qa --filter status=exited)
```

A similar result can be achieved with the simpler-to-remember
`podman container prune`{.inlineCode} command shown here, which removes
(prunes) all stopped containers from the host:

``` {.programlisting .snippet-con}
$ podman container prune
```

**Sorting** is another useful option to produce an ordered output when
listing containers.[]{.sentence-end} The following example shows how to
sort by []{#Chapter_4.xhtml#idx_daf5184a .index-entry
index-entry="running containers:status, handling"}container ID:

``` {.programlisting .snippet-con}
$ podman ps -q --sort id
```

The `podman ps`{.inlineCode} command supports
[]{#Chapter_4.xhtml#idx_2480e6b3 .index-entry
index-entry="running containers:status, viewing"}formatting using a Go
template to produce custom output.[]{.sentence-end} The next example
prints only the container IDs and the commands executed inside them:

``` {.programlisting .snippet-con}
$ podman ps -a --format "{{.ID}}  {{.Command}}" --no-trunc
```

Also, notice the `--no-trunc`{.inlineCode} option is added to avoid
truncating the command output.[]{.sentence-end} This is not mandatory,
but it is useful when we have long commands executed inside the
containers.

If we simply wish to extract the host PID of the process running inside
the running containers, we can run the following example:

``` {.programlisting .snippet-con}
$ podman ps --format "{{ .Pid }}"
```

Instead, if we need to also find out information about the isolated
namespaces, `podman ps`{.inlineCode} can print details about the cloned
namespaces of[]{#Chapter_4.xhtml#idx_b7e8f009 .index-entry
index-entry="running containers:status, handling"} the running
containers.[]{.sentence-end} This is a useful starting point for
advanced troubleshooting and inspection.[]{.sentence-end} You can see
the command being run here:

``` {.programlisting .snippet-con}
$ podman ps --namespace
CONTAINER ID  NAMES                 PID         CGROUPNS    IPC         MNT         NET         PIDNS       USERNS      UTS
f2666ed4a46a  unruffled_hofstadter  437764      4026533088  4026533086  4026533083  4026532948  4026533087  4026532973  4026533085
```

This subsection covered many []{#Chapter_4.xhtml#idx_7eacb767
.index-entry index-entry="running containers:status, viewing"}common
operations to control and view the status of
containers.[]{.sentence-end} In the next section, we will learn how to
pause and resume running containers.

## Pausing and unpausing containers {#Chapter_4.xhtml#h2_124 .heading-2}

This short section covers the `podman pause`{.inlineCode} and
`podman unpause`{.inlineCode} commands.[]{.sentence-end} Despite being a
section related to container status handling, it is interesting to
understand how Podman and the container runtime
[]{#Chapter_4.xhtml#idx_e045ceaa .index-entry
index-entry="running containers:pausing"}leverage cgroups to achieve
specific purposes.

Simply put, the `pause`{.inlineCode} and `unpause`{.inlineCode} commands
have the purpose of pausing and resuming the processes of a running
container.[]{.sentence-end} Now, let\'s[]{#Chapter_4.xhtml#idx_f951a279
.index-entry index-entry="running containers:unpausing"} understand the
difference between `pause`{.inlineCode} and `stop`{.inlineCode} commands
in Podman.

While the `podman stop`{.inlineCode} command simply sends a
`SIGTERM`{.inlineCode}/`SIGKILL`{.inlineCode} signal to the parent
process in the container, the `podman pause`{.inlineCode} command uses
cgroups to pause the process without terminating it.[]{.sentence-end}
When the container is unpaused, the same process is resumed
transparently.

 note
**Tip**

The `pause`{.inlineCode}/`unpause`{.inlineCode} low-level logic is
implemented in the container runtime---for the more curious, this was
the implementation in `crun`{.inlineCode} at the time of writing:

[[https://github.com/containers/crun/blob/7ef74c9330033cb884507c28fd8c267861486633/src/libcrun/cgroup.c#L1894-L1936]{.url}](https://github.com/containers/crun/blob/7ef74c9330033cb884507c28fd8c267861486633/src/libcrun/cgroup.c#L1894-L1936){style="text-decoration: none;"}


The following example demonstrates the
`podman `{.inlineCode}`pause`{.inlineCode} and `unpause`{.inlineCode}
commands.[]{.sentence-end} First, let\'s start a Fedora container that
prints a []{#Chapter_4.xhtml#idx_3d86625b .index-entry
index-entry="running containers:unpausing"}date and time string every
`2`{.inlineCode} seconds in an endless loop, as follows:

``` {.programlisting .snippet-con}
$ podman run --name timer docker.io/library/fedora bash -c "while true; do echo $(date); sleep 2; done"
```

We intentionally leave the []{#Chapter_4.xhtml#idx_05aa9ae8 .index-entry
index-entry="running containers:pausing"}container running in a window
and open a new window/tab to manage its status.[]{.sentence-end} Before
issuing the `pause`{.inlineCode} command, let\'s inspect the PID by
executing the following command:

``` {.programlisting .snippet-con}
$ podman ps --format "{{ .Pid }}" --filter name=timer
816807
```

Now, let\'s pause the running container with the following command:

``` {.programlisting .snippet-con}
$ podman pause timer
```

If we go back to the `timer`{.inlineCode} container, we see that the
output just paused, but the container has not exited.[]{.sentence-end}
The `unpause`{.inlineCode} action seen here will bring it back to life:

``` {.programlisting .snippet-con}
$ podman unpause timer
```

After the `unpause`{.inlineCode} action, the timer container will start
printing date outputs again.[]{.sentence-end} Looking at the PID here,
nothing has changed, as expected:

``` {.programlisting .snippet-con}
$ podman ps --format "{{ .Pid }}" --filter name=timer
816807
```

We can check the cgroups status of the paused/unpaused
container.[]{.sentence-end} In a third tab, open a terminal with a root
shell and access the `cgroupfs`{.inlineCode} controller hierarchy after
replacing the correct container ID, as follows:

``` {.programlisting .snippet-con}
$ sudo -i
$ cd /sys/fs/cgroup/user.slice/user-1000.slice/user@$UID.service/user.slice/libpod-<CONTAINER_ID>.scope/container
```

Now, look at the `cgroup.freeze`{.inlineCode} file
content.[]{.sentence-end} This file holds
a[]{#Chapter_4.xhtml#idx_122fbae0 .index-entry
index-entry="running containers:unpausing"} Boolean value and its state
changes as we pause/unpause the []{#Chapter_4.xhtml#idx_0aa1fd10
.index-entry index-entry="running containers:pausing"}container from 0
to 1 and vice versa.[]{.sentence-end} Try to pause and unpause the
container again to test the changes.

 packt_tip
**Cleanup** **tip**

Since the echo loop was issued with a
`bash -`{.inlineCode}`c`{.inlineCode} command, we need to send a
`SIGKILL`{.inlineCode} signal to the process.[]{.sentence-end} To do
this, we can stop the container and wait for the 10-second timeout, or
simply run a `podman kill`{.inlineCode} command, as follows:

``` {.programlisting .snippet-con-tip}
$ podman kill timer
```


Alternatively, we can even skip the 10-second timeout with the
`--time 0`{.inlineCode} option.

In this subsection, we covered in detail the most common commands to
watch and modify a container\'s status.[]{.sentence-end} We can now move
on to inspect the processes running inside the running containers.

## Inspecting processes inside containers {#Chapter_4.xhtml#h2_125 .heading-2}

When a container is running, processes[]{#Chapter_4.xhtml#idx_3b252505
.index-entry
index-entry="running containers:inside processes, inspecting"} inside it
are isolated at the namespace level, but users still have total control
of the processes running and can inspect their
behavior.[]{.sentence-end} There are many levels of complexity in
process inspection, but Podman offers tools that can speed up this task.

Let\'s start with the `podman top`{.inlineCode} command: this provides a
full view of the processes running inside a container.[]{.sentence-end}
The following example shows the processes running inside an nginx
container:

``` {.programlisting .snippet-con}
$ podman top f2666ed4a46a
USER        PID         PPID        %CPU        ELAPSED          TTY         TIME        COMMAND
root        1           0           0.000       3m26.540290427s  ?           0s          nginx: master process nginx -g daemon off;
nginx       26          1           0.000       3m26.540547429s  ?           0s          nginx: worker process
nginx       27          1           0.000       3m26.540788803s  ?           0s          nginx: worker process
nginx       28          1           0.000       3m26.540914386s  ?           0s          nginx: worker process
nginx       29          1           0.000       3m26.541040023s  ?           0s          nginx: worker process
nginx       30          1           0.000       3m26.541161213s  ?           0s          nginx: worker process
nginx       31          1           0.000       3m26.541297546s  ?           0s          nginx: worker process
nginx       32          1           0.000       3m26.54141773s   ?           0s          nginx: worker process
nginx       33          1           0.000       3m26.541564289s  ?           0s          nginx: worker process
nginx       34          1           0.000       3m26.541685475s  ?           0s          nginx: worker process
nginx       35          1           0.000       3m26.541808977s  ?           0s          nginx: worker process
nginx       36          1           0.000       3m26.541932099s  ?           0s          nginx: worker process
nginx       37          1           0.000       3m26.54205111s   ?           0s          nginx: worker process
```

The result is very similar to the `ps`{.inlineCode} command output
rather than the interactive one produced by the Linux `top`{.inlineCode}
command.

It is possible to apply custom[]{#Chapter_4.xhtml#idx_a3407789
.index-entry
index-entry="running containers:inside processes, inspecting"}
formatting to the output.[]{.sentence-end} The following example only
prints PIDs, commands, and arguments:

``` {.programlisting .snippet-con}
$ podman top f2666ed4a46a pid comm args
PID         COMMAND     COMMAND
1           nginx       nginx: master process nginx -g daemon off;
26          nginx       nginx: worker process
27          nginx       nginx: worker process
28          nginx       nginx: worker process
29          nginx       nginx: worker process
30          nginx       nginx: worker process
31          nginx       nginx: worker process
32          nginx       nginx: worker process
33          nginx       nginx: worker process
34          nginx       nginx: worker process
35          nginx       nginx: worker process
36          nginx       nginx: worker process
37          nginx       nginx: worker process
```

We may need to inspect container processes in greater
detail.[]{.sentence-end} As we discussed earlier, in *[Chapter
1](#Chapter_1.xhtml#h1_14){.chapref}*, *Introduction to Container
Technology*, once a brand new container is started, it will start
assigning PIDs from number `0`{.inlineCode}, while under the hood, the
container engine will map this container\'s PIDs with the real ones on
the host.[]{.sentence-end} So, we can use the output of the
`podman ps --namespace`{.inlineCode} command to extract the process\'s
original PID in the host for a given container.[]{.sentence-end} With
that information, we[]{#Chapter_4.xhtml#idx_d51e5760 .index-entry
index-entry="running containers:inside processes, inspecting"} can
conduct advanced analysis.[]{.sentence-end} The following example shows
how to attach the `strace`{.inlineCode} command, used to inspect
processes\' syscalls, to the process running inside the container:

``` {.programlisting .snippet-con}
$ sudo strace -p <PID>
```

Details about the usage of the `strace`{.inlineCode} command are beyond
the scope of this book.[]{.sentence-end} See `man strace`{.inlineCode}
for more advanced examples and a deeper explanation of the command
options.

Alternatively, the easiest way to gather the PID is using the
`podman top`{.inlineCode} command as described in the next code block:

``` {.programlisting .snippet-con}
$ podman top $CONTAINER_ID pid hpid
```

The `hpid`{.inlineCode} option will print the PID of the process on the
host.

Another useful command that can be easily applied to processes running
inside a container is `pidstat`{.inlineCode}.[]{.sentence-end} Once we
have obtained the PID, we can inspect the resource usage in this way:

``` {.programlisting .snippet-con}
$ pidstat -p <PID> [<interval> <count>]
```

The integers applied at the end represent, respectively, the execution
interval of the command and the number of times it
[]{#Chapter_4.xhtml#idx_e7c8bed9 .index-entry
index-entry="running containers:inside processes, inspecting"}must print
the usage stats.[]{.sentence-end} See `man pidstat`{.inlineCode} for
more usage options.

When a process in a container becomes unresponsive, it is possible to
handle its abrupt termination with the `podman kill`{.inlineCode}
command.[]{.sentence-end} By default, it sends a `SIGKILL`{.inlineCode}
signal to the process inside the container.[]{.sentence-end} The
following example creates an `httpd`{.inlineCode} container and then
kills it:

``` {.programlisting .snippet-con}
$ podman run --name custom-webserver -d docker.io/library/httpd
$ podman kill custom-webserver
```

We can optionally send custom signals (such as `SIGTERM`{.inlineCode} or
`SIGHUP`{.inlineCode}) with the `--signal`{.inlineCode}
option.[]{.sentence-end} Notice that a killed container is not removed
from the host but continues to exist, is stopped, and is in an exited
status.

In *[Chapter 1](#Chapter_1.xhtml#h1_14){.chapref}1*, *Troubleshooting
and Monitoring Containers*, we will again deal with container
troubleshooting and learn how to use advanced tools such as
`nsenter`{.inlineCode} to inspect container processes.[]{.sentence-end}
We\'ll now move on to basic container statistics commands that can be
useful for monitoring the overall resource usage by all containers
running in a system.

## Monitoring container stats {#Chapter_4.xhtml#h2_126 .heading-2}

When multiple containers are []{#Chapter_4.xhtml#idx_bb4dbca8
.index-entry index-entry="running containers:stats, monitoring"}running
on the same host, it is crucial to monitor the amount of CPU, memory,
disk, and network resources they are consuming in a given interval of
time.[]{.sentence-end} The first, simpler command that an administrator
can use is the `podman stats`{.inlineCode} command, shown here:

``` {.programlisting .snippet-con}
$ podman stats
```

Without any options, the command will open a top-like, self-refreshing
window with the stats of all the running containers.[]{.sentence-end}
The default printed values are listed here:

-   `ID`{.inlineCode}: The running container ID
-   `NAME`{.inlineCode}: The running container name
-   `CPU %`{.inlineCode}: The total CPU usage as a percentage
-   `MEM USAGE / LIMIT`{.inlineCode}: Memory usage against a given limit
    (dictated by system capabilities or by cgroups-driven limits)
-   `MEM %`{.inlineCode}: The total memory usage as a percentage
-   `NET IO`{.inlineCode}: Network **input/output** (**I/O**) operations
-   `BLOCK IO`{.inlineCode}: Disk I/O operations
-   `PIDS`{.inlineCode}: The number of PIDs inside the container
-   `CPU TIME`{.inlineCode}: Total consumed CPU time
-   `AVG CPU %`{.inlineCode}: Average CPU usage as a percentage

In case a redirect is needed, it is possible to avoid streaming a
self-refreshing output with the `--no-stream`{.inlineCode} option, as
follows:

``` {.programlisting .snippet-con}
$ podman stats --no-stream
```

Anyway, having a static output of this type is not very useful for
parsing or ingestion.[]{.sentence-end} A better approach is to apply a
JSON or Go template formatter.[]{.sentence-end} The following example
prints out stats in a JSON format:

``` {.programlisting .snippet-con}
$ podman stats --format=json

[
 {
  "id": "d19dbb81a0f3",
  "name": "admiring_bouman",
  "cpu_time": "16.773ms",
  "cpu_percent": "0.01%",
  "avg_cpu": "0.01%",
  "mem_usage": "1.884MB / 475.4MB",
  "mem_percent": "0.40%",
  "net_io": "0B / 796B",
  "block_io": "0B / 0B",
  "pids": "2"
 }
]
```

In a similar way, it is possible to customize the output fields using a
Go template.

 packt_tip
**Good to** **know**

The command-line option for Podman `--format=json`{.inlineCode} works on
most commands.[]{.sentence-end} For example,
`podman ps --format=json`{.inlineCode} for machine-readable output.


The following example only []{#Chapter_4.xhtml#idx_5e7dc89c .index-entry
index-entry="running containers:stats, monitoring"}prints out the
container ID, CPU percentage usage, total memory usage in bytes, and
PIDs:

``` {.programlisting .snippet-con}
$ podman stats -a --no-stream --format "{{ .ID }} {{ .CPUPerc }} {{ .MemUsageBytes }} {{ .PIDs }}"
```

In this section, we have learned how to monitor running containers and
their isolated processes.[]{.sentence-end} The next section shows
[]{#Chapter_4.xhtml#idx_21887b60 .index-entry
index-entry="running containers:stats, monitoring"}how to inspect
container configurations for analysis and troubleshooting.

# Inspecting container information {#Chapter_4.xhtml#h1_127 .heading-1}

A running container exposes a set of []{#Chapter_4.xhtml#idx_8c22b3a5
.index-entry
index-entry="container:information, inspecting"}configuration data and
metadata ready to be consumed.[]{.sentence-end} Podman implements the
`podman inspect`{.inlineCode} command to print all the container
configurations and runtime information.[]{.sentence-end} In its simplest
form, we can simply pass the container ID or name, like this:

``` {.programlisting .snippet-con}
$ podman inspect <Container_ID_or_Name>
```

This command prints a JSON output with all the container
configurations.[]{.sentence-end} For the sake of space, we will list
some of the most notable fields here:

-   `Path`{.inlineCode}: The container entry point
    path.[]{.sentence-end} We will dig deeper into entry points later,
    when we analyze Dockerfiles.
-   `Args`{.inlineCode}: The arguments passed to the entry point that,
    along with `Path`{.inlineCode}, form the command executed in the
    container.
-   `State`{.inlineCode}: The container\'s current state, including
    crucial information such as the executed PID, the common PID, the
    OCI version, and the health check status.
-   `Image`{.inlineCode}: The ID of the image used to run the container.
-   `Name`{.inlineCode}: The container name.
-   `MountLabel`{.inlineCode}: The container mount label for SELinux.
-   `ProcessLabel`{.inlineCode}: The container process label for
    SELinux.
-   `EffectiveCaps`{.inlineCode}: Effective capabilities applied to the
    container.
-   `GraphDriver`{.inlineCode}: The type of storage driver (the default
    is `overlayfs`{.inlineCode}) and a list of overlay upper, lower, and
    merged directories.
-   `Mounts`{.inlineCode}: The actual user-specified mounts in the
    container.
-   `NetworkSettings`{.inlineCode}: The overall container network
    settings, including its internal IP address, exposed ports, and port
    mappings.
-   `Config`{.inlineCode}: The container runtime configuration,
    including environment variables, hostname, command, working
    directory, labels, and annotations.
-   `HostConfig`{.inlineCode}: The host configuration, including
    cgroups\' quotas, network mode, and capabilities.

This is a huge amount of information that, most
[]{#Chapter_4.xhtml#idx_b2f3dc43 .index-entry
index-entry="container:information, inspecting"}of the time, is too much
for our needs.[]{.sentence-end} When we need to extract specific fields,
we can use the `--format`{.inlineCode} option to print only selected
ones.[]{.sentence-end} The following example prints only the host-bound
PID of the process executed inside the container:

``` {.programlisting .snippet-con}
$ podman inspect <ID or Name> --format "{{ .State.Pid }}"
```

The result is in a Go template format.[]{.sentence-end} This allows for
flexibility to customize the output string as we desire.

The `podman inspect`{.inlineCode} command is also useful for
understanding the behavior of the container engine and for gaining
useful information during troubleshooting tasks.

For example, when a container is launched, we learn that the
`resolv.conf`{.inlineCode} file is mounted inside the container from a
path that is defined in the
`{`{.inlineCode}`{ .`{.inlineCode}`ResolvConfPath }}`{.inlineCode}
key.[]{.sentence-end} The target path is
`/run/user/`{.inlineCode}`<`{.inlineCode}`UID`{.inlineCode}`>`{.inlineCode}`/containers/overlay-containers/`{.inlineCode}`<`{.inlineCode}`Container_ID`{.inlineCode}`>`{.inlineCode}`/userdata/resolv.conf`{.inlineCode}
when the container is executed in rootless mode and
`/var/run/containers/storage/overlay-containers/`{.inlineCode}`<`{.inlineCode}`Container_ID`{.inlineCode}`>`{.inlineCode}`/userdata/resolv.conf`{.inlineCode}
when in rootful mode.

Other interesting information is the list of all the merged layers
managed by `overlayfs`{.inlineCode}.[]{.sentence-end} Let\'s try to run
a new container, this time in rootful mode, and find out information
about the merged layers, as follows:

``` {.programlisting .snippet-con}
# podman run --name logger -d docker.io/library/fedora bash -c "while true; do echo test >> /tmp/test.log; sleep 5; done"
```

This container runs a simple loop []{#Chapter_4.xhtml#idx_5cf22c4a
.index-entry index-entry="container:information, inspecting"}that writes
a string to a text file every `5`{.inlineCode} seconds.[]{.sentence-end}
Now, let\'s run a `podman inspect`{.inlineCode} command to find out
information about `MergedDir`{.inlineCode}, which is the directory where
all layers are merged by `overlayfs`{.inlineCode}.[]{.sentence-end} The
related command is illustrated in the following snippet:

``` {.programlisting .snippet-con}
# podman inspect logger --format "{{ .GraphDriver.Data.MergedDir }}"
/var/lib/containers/storage/overlay/4372e7c85d552e84e470223cd5aef0bc0fc69ffdf83f687506dff9bd93b64767/merged
```

Inside this directory, we can find the `/tmp/test.log`{.inlineCode}
file, as indicated here:

``` {.programlisting .snippet-con}
# cat /var/lib/containers/storage/overlay/4372e7c85d552e84e470223cd5aef0bc0fc69ffdf83f687506dff9bd93b64767/merged/tmp/test.log
test
test
test
test
test
[...]
```

We can dig deeper---the `LowerDir`{.inlineCode}
directory[]{#Chapter_4.xhtml#idx_9a22b902 .index-entry
index-entry="container:information, inspecting"} holds a list of the
base image layers, as shown in the following code snippet:

``` {.programlisting .snippet-con}
# podman inspect logger \
--format "{{ .GraphDriver.Data.LowerDir}}"
  
/var/lib/containers/storage/overlay/b0d5c42c12e7b1e896892f1a013ac57b467f2f25545833cf4c5ebc2a5f823845/diff
```

In this example, the base image is made up of only one
layer.[]{.sentence-end} Are we going to find the log file here? Let\'s
have a look:

``` {.programlisting .snippet-con}
# cat /var/lib/containers/storage/overlay/b0d5c42c12e7b1e896892f1a013ac57b467f2f25545833cf4c5ebc2a5f823845/diff/tmp/test.log
cat: /var/lib/containers/storage/overlay/b0d5c42c12e7b1e896892f1a013ac57b467f2f25545833cf4c5ebc2a5f823845/diff/tmp/test.log: No such file or directory
```

We are missing the log file in this layer.[]{.sentence-end} This is
because the `LowerDir`{.inlineCode} directory is not written and
represents the read-only[]{#Chapter_4.xhtml#idx_cf8c934f .index-entry
index-entry="container:information, inspecting"} image
layers.[]{.sentence-end} It is merged with an `UpperDir`{.inlineCode}
directory that is the read-write layer of the
container.[]{.sentence-end} With `podman inspect`{.inlineCode}, we can
find out where it resides, as illustrated here:

``` {.programlisting .snippet-con}
# podman inspect logger --format "{{ .GraphDriver.Data.UpperDir }}"
/var/lib/containers/storage/overlay/27d89046485db7c775b108a80072eafdf9aa63d14ee1205946d74623fc195314/diff
```

The output directory will contain only a bunch of files and directories,
written since the container startup, including the
`/tmp/test.log`{.inlineCode} file, as illustrated in the following code
snippet:

``` {.programlisting .snippet-con}
# cat /var/lib/containers/storage/overlay/4372e7c85d552e84e470223cd5aef0bc0fc69ffdf83f687506dff9bd93b64767/diff/tmp/test.log
test
test
test
test

[...]
```

We can now stop and remove the logger container by running the following
command:

``` {.programlisting .snippet-con}
# podman stop logger && podman rm logger
```

This example was in anticipation of the container storage topic that
will be covered in *[Chapter 5](#Chapter_5.xhtml#h1_133){.chapref}*,
*Implementing Storage for the Container\'s Data*.[]{.sentence-end} The
`overlayfs`{.inlineCode} mechanisms, with the lower, upper, and merged
directory concepts, will be analyzed in more detail.

In this section, we learned how[]{#Chapter_4.xhtml#idx_73097530
.index-entry index-entry="container:information, inspecting"} to inspect
running containers and collect runtime information and
configurations.[]{.sentence-end} The next section is going to cover best
practices for capturing logs from containers.

# Capturing logs from containers {#Chapter_4.xhtml#h1_128 .heading-1}

As described earlier in this chapter, containers
[]{#Chapter_4.xhtml#idx_68f200fd .index-entry
index-entry="container:logs, capturing"}are made of one or more
processes that can fail, printing errors and describing their current
state in a log file.[]{.sentence-end} But where are these logs
[]{#Chapter_4.xhtml#idx_379f3f2b .index-entry
index-entry="logs:capturing, from containers"}stored?

Well, of course, a process in a container could write its log messages
inside a file somewhere in a temporary filesystem that the container
engine has made available to it (if any).[]{.sentence-end} But what
about a read-only filesystem or any permission constraints in the
running container?

A container\'s best practice for exposing relevant logs outside the
container\'s shield actually leverages []{#Chapter_4.xhtml#idx_f1dfd452
.index-entry index-entry="standard error (STDERR)"}the use of
standard[]{#Chapter_4.xhtml#idx_dd96b2bb .index-entry
index-entry="standard output (STDOUT)"} streams: **standard output**
(`STDOUT`{.inlineCode}) and **standard error** (`STDERR`{.inlineCode}).

 note
**Good to** **know**

Standard streams are communication[]{#Chapter_4.xhtml#idx_1f169e22
.index-entry index-entry="standard streams"} channels interconnected to
a running process in an OS.[]{.sentence-end} When a program is run
through an interactive shell, these streams are then directly connected
to the user\'s running terminal to let input, output, and error flow
between the terminal and the process, and vice versa.


Depending on the options we use for running a brand-new container,
Podman will act appropriately by attaching the `STDIN`{.inlineCode},
`STDOUT`{.inlineCode}, and `STDERR`{.inlineCode} standard streams to a
local file for storing the logs.[]{.sentence-end} For
Linux[]{#Chapter_4.xhtml#idx_d2395eb3 .index-entry
index-entry="logs:capturing, from containers"} distributions that use
systemd as an init manager, logs are stored in journald by default.

In *[Chapter 3](#Chapter_3.xhtml#h1_83){.chapref}*, *Running* *the*
*First Container*, we saw how to run a container in the background,
detaching from a running container.[]{.sentence-end} We used the
`-d`{.inlineCode} option to start a container in *detached* mode through
the `podman run`{.inlineCode} command, as illustrated here:

``` {.programlisting .snippet-con}
$ podman run -d -i -t docker.io/library/httpd
```

With the previous command, we are instructing Podman to start a
container in detached mode (`-d`{.inlineCode}), with a pseudo-terminal
attached to the `STDIN`{.inlineCode} stream (`-t`{.inlineCode}), keeping
the standard input stream open even if there is no terminal
[]{#Chapter_4.xhtml#idx_678a63c8 .index-entry
index-entry="container:logs, capturing"}attached yet
(`-i`{.inlineCode}).

The standard Podman behavior was to attach to `STDOUT`{.inlineCode} and
`STDERR`{.inlineCode} streams and store any container\'s published data
in a log file on the host filesystem.

In the Fedora Linux distribution, starting from version 35, the
maintainers decided to switch from `k8s-file`{.inlineCode} to
`journald`{.inlineCode}.[]{.sentence-end} In this case, you could look
for the logs directly using the `journalctl`{.inlineCode} command-line
utility.

If you want to take a look at the default `log_driver`{.inlineCode}
field, you can look in the following path:

``` {.programlisting .snippet-con}
# grep ^log_driver /usr/share/containers/containers.conf
log_driver = "journald"
```

If we are working with Podman as a root user, we can take a look at the
log file available on the host system, executing the following steps:

1.  First, we need to start our container and take note of the name
    assigned by Podman.[]{.sentence-end} The code to accomplish this is
    shown in the following snippet:

    ``` {.programlisting .snippet-con-one}
    # podman run -d -i -t docker.io/library/httpd
    a068c7db7ff996fc74499909964c6dc6ce22a0d184659ffe5f650264cb5cb553
      
    ```

2.  After that, we can take []{#Chapter_4.xhtml#idx_c5764c2f
    .index-entry index-entry="container:logs, capturing"}our
    container\'s name, as follows:

    ``` {.programlisting .snippet-con-one}
    # podman ps
    CONTAINER ID  IMAGE                           COMMAND           CREATED         STATUS         PORTS       NAMES
    a068c7db7ff9  docker.io/library/httpd:latest  httpd-foreground  15 minutes ago  Up 15 minutes  80/tcp      cranky_wu
    ```

3.  Finally, we can check the logs of our running container by asking
    `journald`{.inlineCode} with the right option,
    specifying[]{#Chapter_4.xhtml#idx_a7e1ce86 .index-entry
    index-entry="logs:capturing, from containers"} the container\'s
    name:

    ``` {.programlisting .snippet-con-one}
    # journalctl --user -t cranky_wu
    Oct 10 09:12:42 localhost.localdomain cranky_wu[13287]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.88.0.3. Set the 'ServerName' directiv>
    Oct 10 09:12:42 localhost.localdomain cranky_wu[13287]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.88.0.3. Set the 'ServerName' directiv>
    Oct 10 09:12:42 localhost.localdomain cranky_wu[13287]: [Thu Oct 10 09:12:42.379839 2024] [mpm_event:notice] [pid 1:tid 1] AH00489: Apache/2.4.62 (Unix) configured -- resuming normal operat>
    Oct 10 09:12:42 localhost.localdomain cranky_wu[13287]: [Thu Oct 10 09:12:42.380944 2024] [core:notice] [pid 1:tid 1] AH00094: Command line: 'httpd -D FOREGROUND'
      
    ```

Does this mean that we need to[]{#Chapter_4.xhtml#idx_9e50938b
.index-entry index-entry="container:logs, capturing"} do all this
complex procedure every time we need to analyze the logs of our
containers? Of course not!

Podman has a `podman logs`{.inlineCode} built-in command that can easily
discover, grab, and print the latest container logs for
us.[]{.sentence-end} Considering[]{#Chapter_4.xhtml#idx_2694b1d1
.index-entry index-entry="logs:capturing, from containers"} the previous
example, we can easily check the logs of our running container by
executing the following command:

``` {.programlisting .snippet-con}
# podman logs cranky_wu
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.88.0.3. Set the 'ServerName' directive globally to suppress this message
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.88.0.3. Set the 'ServerName' directive globally to suppress this message
[Thu Oct 10 09:12:42.379839 2024] [mpm_event:notice] [pid 1:tid 1] AH00489: Apache/2.4.62 (Unix) configured -- resuming normal operations
[Thu Oct 10 09:12:42.380944 2024] [core:notice] [pid 1:tid 1] AH00094: Command line: 'httpd -D FOREGROUND'
```

We can also get the short ID for our running container and pass this ID
to the `podman logs`{.inlineCode} command, as follows:

``` {.programlisting .snippet-con}
# podman ps
CONTAINER ID  IMAGE                           COMMAND           CREATED         STATUS         PORTS       NAMES
a068c7db7ff9  docker.io/library/httpd:latest  httpd-foreground  19 minutes ago  Up 19 minutes  80/tcp      cranky_wu

# podman logs --tail 2 a068c7db7ff9
[Thu Oct 10 09:12:42.379839 2024] [mpm_event:notice] [pid 1:tid 1] AH00489: Apache/2.4.62 (Unix) configured -- resuming normal operations
[Thu Oct 10 09:12:42.380944 2024] [core:notice] [pid 1:tid 1] AH00094: Command line: 'httpd -D FOREGROUND'
```

In the previous command, we also []{#Chapter_4.xhtml#idx_55729319
.index-entry index-entry="container:logs, capturing"}used a nice option
of the `podman logs`{.inlineCode} command: the `--tail`{.inlineCode}
option that let us output only the latest needed rows of the
container\'s log.[]{.sentence-end} In our case, we requested the latest
two.

 packt_tip
**Good to** **know**

Most of the commands in Podman accept the `--latest`{.inlineCode}
command-line option.[]{.sentence-end} This causes the command to work on
the most recently created container, which is useful for situations like
this.[]{.sentence-end} If you only have one container,
`podman logs --latest`{.inlineCode} will always give the right result
without having to look up the ID.


We are now ready to move []{#Chapter_4.xhtml#idx_bdcf6f72 .index-entry
index-entry="logs:capturing, from containers"}on to the next section,
where we will discover another useful command.

# Executing processes into a running container {#Chapter_4.xhtml#h1_129 .heading-1}

In the *Podman daemonless architecture* section of *[Chapter
2](#Chapter_2.xhtml#h1_52){.chapref}*, *Comparing Podman and Docker*, we
talked about the fact that Podman, as with any other container engine,
leverages the Linux namespace functionality to correctly isolate running
containers from each other and from the OS host as well.

So, just because Podman creates a []{#Chapter_4.xhtml#idx_35056981
.index-entry
index-entry="container:processes, executing into running container"}brand-new
namespace for every running container, it should not be a surprise that
we can attach to the same Linux[]{#Chapter_4.xhtml#idx_192f2777
.index-entry index-entry="running containers:processes, executing"}
namespace of a running container, executing other processes just as in a
full operating environment.

Podman gives us the ability to execute a process in a running container
through the `podman exec`{.inlineCode} command.

Once executed, this command will internally find the right Linux
namespace to which the target running container is
attached.[]{.sentence-end} Having found the Linux namespace, Podman will
execute the respective process, passed as an argument to the
`podman exec`{.inlineCode} command, attaching it to the target Linux
namespace.[]{.sentence-end} The final process will be in the same
environment as the original process companion, and it will be able to
interact with it; this preserves the security of the namespace and
container.

To understand how this works in practice, we can consider the following
example, whereby we will first run a container and then execute a
process beside the existing processes:

``` {.programlisting .snippet-con}
# podman run -d -i -t docker.io/library/httpd
e768bf9723687194329b6d6854605849529c79f1d191d3e7139ed5f4e4ae04eb

# podman exec -ti e768bf9723687194329b6d6854605849529c79f1d191d3e7139ed5f4e4ae04eb /bin/bash
root@e768bf972368:/usr/local/apache2# ps aux
bash: ps: command not found
root@e768bf972368:/usr/local/apache2# ls -l /proc/*/exe
[omitted output]
lrwxrwxrwx. 1 root root 0 Oct 10 09:36 /proc/1/exe -> /usr/local/apache2/bin/httpd

...
```

As you can see from the previous commands, we grabbed the container ID
provided by Podman once the container was started, and we passed it to
the `podman exec`{.inlineCode} command as an argument.

The `podman exec`{.inlineCode} command
could[]{#Chapter_4.xhtml#idx_00523267 .index-entry
index-entry="container:processes, executing into running container"} be
really useful for troubleshooting, testing, and working with an existing
container.[]{.sentence-end} In the preceding example, we attached an
interactive terminal running the Bash console, and we launched the
`ps`{.inlineCode} command for inspecting the running processes available
in the current Linux namespace assigned to the
container.[]{.sentence-end} However, the `ps`{.inlineCode} command was
not available, which means that maybe[]{#Chapter_4.xhtml#idx_3148b299
.index-entry index-entry="running containers:processes, executing"} the
developers who built this container image decided not to include the
`procps`{.inlineCode} package to shrink the size of the
image.[]{.sentence-end} So, to overcome the absence of the
`ps`{.inlineCode} command, we just looked in the `/proc`{.inlineCode}
filesystem; luckily, everything is a file in Linux, isn\'t it? The
alternative could be to run a package manager inside the container and
install the package containing the `ps`{.inlineCode} command.

The `podman exec`{.inlineCode} command has many options available,
similar to the ones provided by the `podman run`{.inlineCode}
command.[]{.sentence-end} As you saw from the previous example, we used
the option for getting a pseudo-terminal attached to the
`STDIN`{.inlineCode} stream (`-t`{.inlineCode}), keeping the standard
input stream open even if there is no terminal attached yet
(`-i`{.inlineCode}).

For more details on the available options, we can check the manual with
the respective command, as illustrated here:

``` {.programlisting .snippet-con}
# man podman exec
```

We are moving forward in our journey through the container management
world, and in the next section, we will also take a look at some of the
capabilities that Podman offers to enable containerized workloads in the
Kubernetes container orchestration world.

# Running containers in pods {#Chapter_4.xhtml#h1_130 .heading-1}

As we mentioned in *[Chapter 2](#Chapter_2.xhtml#h1_52){.chapref}*,
*Comparing Podman* *and* *Docker*, Podman offers capabilities to easily
start adopting some basic []{#Chapter_4.xhtml#idx_8047fc6a .index-entry
index-entry="running containers:in pods"}concepts of the de facto
container orchestrator named[]{#Chapter_4.xhtml#idx_87c753ea
.index-entry index-entry="Kubernetes (k8s)"} Kubernetes (also sometimes
referred to as `k8s`{.inlineCode}).

The pod concept was introduced with []{#Chapter_4.xhtml#idx_dc18eae4
.index-entry index-entry="pods:running container"}Kubernetes and
represents the smallest execution unit in a Kubernetes
cluster.[]{.sentence-end} With Podman, users can create empty pods and
then run containers inside them easily.

Grouping two or more containers inside a single pod can have many
benefits, such as the following:

-   Sharing the same network namespace, IP address included
-   Sharing the same storage volumes for storing persistent data
-   Sharing the same configurations
-   Managing the containers together (starting, stopping, restarting as
    a single unit)

In addition, placing two or more containers in the
[]{#Chapter_4.xhtml#idx_ed0d8d01 .index-entry
index-entry="inter-process communication (IPC)"}same pod will actually
enable them to share the same **inter-process communication** (**IPC**)
Linux namespace.[]{.sentence-end} This could be really useful for
applications that need to communicate with each other using shared
memory.

The simplest way to create a pod and start working with it is to use
this command:

``` {.programlisting .snippet-con}
# podman pod create --name myhttp
7a566073adb31df12286bfa82586d82b5004396ff10a2235d9d7cf7e7a49cb9f
# podman pod ps
POD ID        NAME        STATUS      CREATED         INFRA ID      # OF CONTAINERS
7a566073adb3  myhttp      Created     13 seconds ago  b5483a84d10b  1
```

As shown in the previous example, we create a new pod named
`myhttp`{.inlineCode} and then check the status of the pod on our host
system: there is just one pod in a `created`{.inlineCode} state.

We can now start the pod as follows and check what will happen:

``` {.programlisting .snippet-con}
# podman pod start myhttp
myhttp
# podman pod ps
POD ID        NAME        STATUS      CREATED             INFRA ID      # OF CONTAINERS
7a566073adb3  myhttp      Running     About a minute ago  b5483a84d10b  1
```

The pod is now running, but []{#Chapter_4.xhtml#idx_e055b5b5
.index-entry index-entry="running containers:in pods"}what is Podman
actually running? We created an empty pod without
[]{#Chapter_4.xhtml#idx_500c8933 .index-entry
index-entry="pods:running container"}containers inside! Let\'s take a
look at the running container by executing the `podman ps`{.inlineCode}
command, as follows:

``` {.programlisting .snippet-con}
# podman ps
CONTAINER ID  IMAGE                                    COMMAND     CREATED        STATUS             PORTS       NAMES
b5483a84d10b  localhost/podman-pause:5.2.2-1724198400              2 minutes ago  Up About a minute              7a566073adb3-infra
```

The `podman ps`{.inlineCode} command is showing a running container with
an image named `pause`{.inlineCode}.[]{.sentence-end} This container is
run by Podman by default as an `infra`{.inlineCode}
container.[]{.sentence-end} This kind of container does nothing---it
just holds the namespace and lets the container engine connect to any
other running container inside the pod.

Having demystified the role of this special container inside our pods,
we can now take a brief look at the steps required to start a
multi-container pod.

First of all, let\'s start by running a new container inside the
existing pod we created in the previous example, as follows:

``` {.programlisting .snippet-con}
# podman run --pod myhttp -d -i -t docker.io/library/httpd
9c564757d9b34f6c3eb9e14672b9e6d8e639fb91707143a44356cde6ee6a4a4b
```

Then, we can check if the existing pod has updated the number of
containers it contains, as illustrated in the following code snippet:

``` {.programlisting .snippet-con}
# podman pod ps
POD ID        NAME        STATUS      CREATED        INFRA ID      # OF CONTAINERS
7a566073adb3  myhttp      Running     3 minutes ago  b5483a84d10b  2
```

Finally, we can ask Podman for a list of running containers with the
associated pod name, as follows:

``` {.programlisting .snippet-con}
# podman ps -p
CONTAINER ID  IMAGE                                    COMMAND           CREATED         STATUS         PORTS       NAMES               POD ID        PODNAME
b5483a84d10b  localhost/podman-pause:5.2.2-1724198400                    3 minutes ago   Up 2 minutes               7a566073adb3-infra  7a566073adb3  myhttp
9c564757d9b3  docker.io/library/httpd:latest           httpd-foreground  37 seconds ago  Up 37 seconds              goofy_nobel         7a566073adb3  myhttp
```

As you can see, the two containers[]{#Chapter_4.xhtml#idx_0d1b89b0
.index-entry index-entry="running containers:in pods"} running are both
associated with the pod []{#Chapter_4.xhtml#idx_8b2aad66 .index-entry
index-entry="pods:running container"}named `myhttp`{.inlineCode}!

 packt_tip
**Important** **note**

Please consider periodically cleaning up the lab environment after
completing all the examples contained in this chapter.[]{.sentence-end}
This could help you save resources and avoid any errors when moving on
to the next chapter\'s examples.[]{.sentence-end} The two commands that
could help here are `podman rm -`{.inlineCode}`all --force`{.inlineCode}
and `podman system reset`{.inlineCode}, which actually clean up the
Podman environment and reset the Podman configuration to its default.

For this reason, you can refer to the code provided in the
`AdditionalMaterial`{.inlineCode} folder in the book\'s GitHub
repository:
[[https://github.com/PacktPublishing/Podman-for-DevOps-Second-Edition]{.url}](https://github.com/PacktPublishing/Podman-for-DevOps-Second-Edition){style="text-decoration: none;"}.


With the same approach, we can add more and more containers to the same
pod, letting them share all the data we described before.

Please note that placing containers in the same pod can be beneficial in
some cases, but this represents an anti-pattern for the container
technology.[]{.sentence-end} In fact, as mentioned before, Kubernetes
considers the pod the smallest computing unit to run on top of the
distributed nodes\' part of one cluster.[]{.sentence-end} This means
that once you group two or more containers under the same pod, they will
be executed together on the same node, and the orchestrator cannot
balance or distribute their workload on[]{#Chapter_4.xhtml#idx_44ce2507
.index-entry index-entry="running containers:in pods"} multiple
machines.

We will explore more about Podman\'s features that can enable you to
enter the container orchestration world
through[]{#Chapter_4.xhtml#idx_ae2d66e1 .index-entry
index-entry="pods:running container"} Kubernetes in the following
chapters!

# Summary {#Chapter_4.xhtml#h1_131 .heading-1}

In this chapter, we started developing experience in managing
containers, starting with container images, then working with running
containers.[]{.sentence-end} Once our containers were running, we also
explored the various commands available in Podman to inspect and check
the logs and troubleshoot our containers.[]{.sentence-end} The
operations needed to monitor and look after running containers are
really important for any container administrator.[]{.sentence-end}
Finally, we also took a brief look at the Kubernetes concepts available
in Podman that let us group two or more containers under the same Linux
namespace.[]{.sentence-end} All the concepts and examples we just went
through will help us start our experience as a system administrator for
container technologies.

We are now ready to explore another important topic in the next chapter:
managing storage for our containers!

# Join us on Discord {#Chapter_4.xhtml#h1_132 .heading-1}

For discussions around the book and to connect with your peers, join us
on Discord at
[[packt.link/discordcloud]{.url}](https://packt.link/discordcloud){style="text-decoration: none;"}
or scan the QR code below:

![Image](images/B31467_4_1.png){style="width:25%;"}


[]{#Chapter_5.xhtml}

 {.section .chapter}