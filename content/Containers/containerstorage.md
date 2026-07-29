# 5 {#Chapter_5.xhtml#h1_133 .chapterNumber}

# Implementing Storage for the Container\'s Data {#Chapter_5.xhtml#h1_134 .chapterTitle}

In the previous chapters, we explored how to run and manage our
containers using Podman, but we will soon come to realize in this
chapter that these operations aren\'t useful in certain scenarios where
the applications included in our containers need to store data in a
persistent mode.[]{.sentence-end} Containers are ephemeral by default,
and this is one of their main features, as we described in the first
chapter of this book; for this reason, we need a way to attach
persistent storage to a running container to preserve the container\'s
important data.

In this chapter, we\'re going to cover the following main topics:

-   Why does storage matter for containers?
-   Containers\' storage features
-   Copying files in and out of a container
-   Attaching host storage to a container

# Technical requirements {#Chapter_5.xhtml#h1_135 .heading-1}

Before proceeding with the chapter\'s hands-on tutorials, a machine with
a working Podman installation is required.[]{.sentence-end} As stated in
*[Chapter 3](#Chapter_3.xhtml#h1_83){.chapref}*, all the examples in the
book are executed on a Fedora 40 system or later, but can be reproduced
on your operating system of choice.

Finally, a good understanding of the topics covered in *[Chapter
4](#Chapter_4.xhtml#h1_114){.chapref}* is useful in terms of being able
to easily grasp concepts regarding OCI images and container execution.

# Why does storage matter for containers? {#Chapter_5.xhtml#h1_136 .heading-1}

Before[]{#Chapter_5.xhtml#idx_24d60cae .index-entry
index-entry="containers:storage, significance"} moving forward in the
chapter and answering this interesting question, we need to distinguish
between two kinds of storage for containers:

-   *External storage* attached to []{#Chapter_5.xhtml#idx_bb9f14ab
    .index-entry index-entry="containers:external storage"}running
    containers to store data, making it persistent to a container\'s
    restart
-   *Underlying storage* for[]{#Chapter_5.xhtml#idx_791577e7
    .index-entry index-entry="containers:underlying storage"} the root
    filesystems of our containers and container images

Talking about external storage, as we described in *[Chapter
1](#Chapter_1.xhtml#h1_14){.chapref}*, containers are stateless,
ephemeral, and often have a read-only filesystem.[]{.sentence-end} This
is because the theory behind the technology states that containers
should be used for spawning scalable and distributed applications that
have to scale horizontally instead of vertically.

Scaling an application horizontally means that in case we require
additional resources for our running services, we will not increase CPU
or RAM for a single running container, but we will instead launch a
brand new container that will handle the incoming requests along with
the existing container.[]{.sentence-end} This is the same well-known
paradigm adopted in the public cloud.[]{.sentence-end} The container, in
principle, should be ephemeral because any additional copy of the
existing container image should be able to run whenever needed to
support the currently running service.

Of course, exceptions exist, and it could happen that a running
container cannot be scaled horizontally or that it simply needs to share
configurations, cache, or any other data relevant to other copies of the
same container images at startup time or during runtime.

Let\'s understand this with the help of a real-life
example.[]{.sentence-end} Using a car-sharing service to get a new car
for every destination inside a city can be a useful and smart way to
move around without worrying about parking fees, fuel, and other
things.[]{.sentence-end} However, at the same time, this service cannot
allow you to store or leave your stuff inside a parked
car.[]{.sentence-end} Therefore, when using a car-sharing service, we
can unpack our stuff once we get into a car, but we must pack it back
before we leave that car.[]{.sentence-end} The same applies similarly to
containers, where we must attach to them some storage for letting our
container write data down, but then, once our container stops, we
should[]{#Chapter_5.xhtml#idx_d1fe81a7 .index-entry
index-entry="containers:storage, significance"} detach that storage so
that a brand-new container can use it when needed.

Here\'s another more technical example: Let\'s consider a standard
three-tier application with a web, a backend, and a database
service.[]{.sentence-end} Every layer of this application may need
storage, which it will use in a variety of ways.[]{.sentence-end} The
web service may need a place to save a cache, store rendered web pages,
some customized images at runtime, and so on.[]{.sentence-end} The
backend service will need a place to store configuration and
synchronization data between the other running backend services, if any,
and so on.[]{.sentence-end} The database service will surely need a
place to store the data.

Storage is often associated with low-level infrastructure, but in a
container, the storage becomes important even for developers, who should
plan where to attach the storage and the features needed for their
application.

If we extend the topic to container orchestration, then the storage
inherits a strategic role because it should be as elastic and feasible
as the Kubernetes orchestrator that we might use it
with.[]{.sentence-end} The container storage in this case should become
more like software-defined storage -- able to provide storage resources
in a self-service way to developers, and to containers in general.

Although this book will talk about local storage, it\'s important to
note that this is not enough for the Kubernetes orchestrator because
containers should be portable from one host to another, depending on the
availability and scaling rules defined.[]{.sentence-end} This is where
software-defined storage could be the solution!

As we can deduce from the previous examples, external storage matters in
containers.[]{.sentence-end} The usage may vary depending on the running
application inside our container, but it is required.[]{.sentence-end}
At the same time, another key role is driven by the underlying container
storage that is responsible for handling the correct storage of
containers and the container images\' root filesystem.[]{.sentence-end}
Choosing the right, stable, and performing underlying local storage will
ensure better and correct management of our containers.

So, let\'s first explore a bit of the theory of container storage and
then discuss how to work with it.

# Containers\' storage features {#Chapter_5.xhtml#h1_137 .heading-1}

Before going []{#Chapter_5.xhtml#idx_40ae3bc3 .index-entry
index-entry="containers:storage features"}into a real example and use
cases, we should first dig into the main differences between container
[]{#Chapter_5.xhtml#idx_aec39b54 .index-entry
index-entry="container storage interface (CSI)"}storage and a
**container storage interface** (**CSI**).

Container storage, previously referred to as *underlying container
storage*, is responsible for handling container
images[]{#Chapter_5.xhtml#idx_fe99cf5b .index-entry
index-entry="copy-on-write (COW) filesystems"} on **copy-on-write**
(**COW**) filesystems.[]{.sentence-end} Container images need to be
managed until a container engine is instructed to run them, so we need a
way to store the image until it is run.[]{.sentence-end} That\'s the
role of container storage.

Once we start using an orchestrator such as Kubernetes, CSI instead is
responsible for providing container block or file storage that
containers need to write data to.

In the next section, we will concentrate on container storage and its
configuration.[]{.sentence-end} Later, we will talk about external
storage for containers and the options we have in Podman to expose the
host\'s local storage to the running containers.

A great innovation introduced with Podman
is[]{#Chapter_5.xhtml#idx_464f00e6 .index-entry
index-entry="containers/storage project:reference link"} the
`containers/storage`{.inlineCode} project
([[https://github.com/containers/storage]{.url}](https://github.com/containers/storage){style="text-decoration: none;"}),
a great way to share an underlying common method for accessing container
storage on a host.[]{.sentence-end} With the arrival of Docker, we were
forced to pass through the Docker daemon to interact with container
storage.[]{.sentence-end} With no other way to directly interact with
the underlying storage, the Docker daemon just hid it from the user as
well as the system administrator.

With the `containers/storage`{.inlineCode} project, we now have an easy
way to use multiple tools for analyzing, managing, or working with
container storage at the same time.[]{.sentence-end} The configuration
of this low-level piece of software is so important for Podman and its
companion tools, such as Buildah, Skopeo, and CRI-O, and can be
inspected or edited through its configuration file, available at
`/etc/containers/storage.conf`{.inlineCode}.

Looking at the configuration file, we can easily discover that we can
change a lot of options in terms of how our containers interact with the
underlying storage.[]{.sentence-end} Let\'s inspect the most important
option -- the storage driver.

## Storage driver {#Chapter_5.xhtml#h2_138 .heading-2}

The []{#Chapter_5.xhtml#idx_2c4f0e42 .index-entry
index-entry="storage driver"}configuration file,
as[]{#Chapter_5.xhtml#idx_69062613 .index-entry
index-entry="containers:storage driver"} one of its first options,
allows choosing the default COW container storage
driver.[]{.sentence-end} The configuration file in the current version,
at the time of writing this book, supports the following COW drivers:

-   `overlay`{.inlineCode}
-   `vfs`{.inlineCode}
-   `btrfs`{.inlineCode}
-   `zfs`{.inlineCode}

These are also often referred[]{#Chapter_5.xhtml#idx_e41e78a8
.index-entry index-entry="graph drivers"} to as **graph drivers**
because most of them organize the layers they handle in a graph
structure.

When using Fedora 40 and Podman 5.2.2, the container storage
configuration file ships with **overlay** set as the default driver.

Before diving into the other available options and, later, the practical
examples contained in this chapter, let\'s take a closer look at how one
of these COW filesystem drivers works.

The OverlayFS union filesystem has been present in a Linux kernel since
version 3.18.[]{.sentence-end} It is usually enabled by default and
activated dynamically once a mount is initiated with this
filesystem.[]{.sentence-end} The mechanism behind this filesystem is
really simple but powerful -- it allows multiple directory trees to be
overlaid on another, storing only the differences, but showing the
latest updated, *squashed* tree of directories.

Usually, in the world of containers, we start using a read-only
filesystem, adding one or more layers, read-only again, until a running
container will use this bunch of squashed layers as its root
filesystem.[]{.sentence-end} This is where the last read-write layer
will be created as an overlay of the others.

Let\'s see what happens under the hood once we pull down a brand-new
container image with Podman:

 packt_tip
**Important** **note**

If you wish to proceed with testing the following example on your test
machine, ensure that you remove any running containers and container
images to easily match the image with the layers that Podman will
download for us.


``` {.programlisting .snippet-con}
# podman pull docker.io/library/httpd:latest
Trying to pull docker.io/library/httpd:latest......
Getting image source signatures
Copying blob 6bd9d3710aae done   |
Copying blob 4f4fb700ef54 skipped: already exists 
Copying blob a480a496ba95 done   |
Copying blob 3a2663e66670 done   |
Copying blob dbde712f81fb done   |
Copying blob 867b2ea3628d done   |
Copying config 1bcf11fa15 done   |
Writing manifest to image destination
1bcf11fa154f23987201bd92a75bf75e3507fc49f415d5dfe35887d1be3fd596
```

We can see []{#Chapter_5.xhtml#idx_1f4d43e3 .index-entry
index-entry="containers:storage driver"}from the previous command output
that multiple layers have been downloaded.[]{.sentence-end} That\'s
because the container image we pulled down is composed of many layers.

Now, we can start[]{#Chapter_5.xhtml#idx_1ef6e2b6 .index-entry
index-entry="storage driver"} inspecting just the downloaded
layers.[]{.sentence-end} First of all, we have to locate the right
directory, which we can search for inside the configuration
file.[]{.sentence-end} Alternatively, we can use an easier
way.[]{.sentence-end} Podman has a command dedicated to displaying its
running configuration and other useful information --
`podman info`{.inlineCode}.[]{.sentence-end} Let\'s see how it works:

``` {.programlisting .snippet-con}
# podman info | grep -A17 "store:"
store:
  configFile: /usr/share/containers/storage.conf
  containerStore:
    number: 4
    paused: 0
    running: 0
    stopped: 4
  graphDriverName: overlay
  graphOptions:
    overlay.imagestore: /usr/lib/containers/storage
    overlay.mountopt: nodev,metacopy=on
  graphRoot: /var/lib/containers/storage
  graphRootAllocated: 4212109312
  graphRootUsed: 1196261376
  graphStatus:
    Backing Filesystem: btrfs
    Native Overlay Diff: "false"
    Supports d_type: "true": "true"
    Supports shifting: "true"
    Supports volatile: "true"
    Using metacopy: "true": "true"
  imageCopyTmpDir: /var/tmptmp
  imageStore::
    number: 4
  runRoot: /run/containers/storage: /run/containers/storage
  transientStore: false: false
  volumePath: /var/lib/containers/storage/volumes: /var/lib/containers/storage/volumes
```

To reduce[]{#Chapter_5.xhtml#idx_5d4550a7 .index-entry
index-entry="containers:storage driver"} the output of the
`podman info`{.inlineCode} command, we used the `grep`{.inlineCode}
command[]{#Chapter_5.xhtml#idx_85da115c .index-entry
index-entry="storage driver"} to only match the `store`{.inlineCode}
section that contains the current configuration in place for container
storage.[]{.sentence-end} As an alternative, you can also use the
`jq`{.inlineCode} command utility to filter the output requested in JSON
format with the following command:

``` {.programlisting .snippet-con}
$ podman info --format json | jq .store
```

As we can see, the driver used is *overlay*, and the root directory to
search our layers is reported as the `graphRoot`{.inlineCode} directory:
`/var/lib/containers/storage`{.inlineCode}; for rootless containers, the
equivalent is
`$HOME/.local/share/containers/storage`{.inlineCode}.[]{.sentence-end}
We also have other paths reported, but we will talk about these later in
this section.[]{.sentence-end} The keyword `graph`{.inlineCode} is a
term derived[]{#Chapter_5.xhtml#idx_17904bbc .index-entry
index-entry="storage driver"} from[]{#Chapter_5.xhtml#idx_78853128
.index-entry index-entry="containers:storage driver"} the category of
drivers we just introduced earlier.

Let\'s take a look at that directory to see what the actual content is:

``` {.programlisting .snippet-con}
# cd /var/lib/containers/storage
# ls
db.sql  defaultNetworkBackend  libpod  overlay  overlay-containers  overlay-images  overlay-layers  secrets  storage.lock  tmp  userns.lock  volumes
```

We have several directories available whose names are pretty
self-explanatory.[]{.sentence-end} The ones we are looking for are as
follows:

-   `overlay-images`{.inlineCode}: This contains the metadata of the
    container images downloaded
-   `overlay-layers`{.inlineCode}: This contains the archives for all
    the layers of every container image
-   `overlay`{.inlineCode}: This is the directory containing the
    unpacked layers of every container image

Let\'s check the content of the first directory,
`overlay-images`{.inlineCode}:

``` {.programlisting .snippet-con}
# ls -l overlay-images/
total 8
drwx------. 1 root root  646 Oct 19 13:44 1bcf11fa154f23987201bd92a75bf75e3507fc49f415d5dfe35887d1be3fd596  646 Oct 19 13:44 1bcf11fa154f23987201bd92a75bf75e3507fc49f415d5dfe35887d1be3fd596
-rw-------. 1 root root 1505 Oct 19 13:44 images.json 1505 Oct 19 13:44 images.json
-rw-r--r--. 1 root root   64 Oct 19 13:44 images.lock   64 Oct 19 13:44 images.lock
```

As we can imagine, in this directory, we can find the metadata of the
only container image we pulled down and, in the directory with a very
long ID, we will find the manifest file describing the layers that
[]{#Chapter_5.xhtml#idx_68671dc3 .index-entry
index-entry="storage driver"}make up our container image.

Let\'s now[]{#Chapter_5.xhtml#idx_264cd21b .index-entry
index-entry="containers:storage driver"} check the content of the second
directory, `overlay-layers`{.inlineCode}:

``` {.programlisting .snippet-con}
# ls -l overlay-layers/
total 388
-rw-------. 1 root root    412 Oct 19 13:44 7555495f10f71578f8bd7904214940724f63e98660a813f4fa6aa856dd22d8b7.tar-split.gz root    412 Oct 19 13:44 7555495f10f71578f8bd7904214940724f63e98660a813f4fa6aa856dd22d8b7.tar-split.gz
-rw-------. 1 root root 272894 Oct 19 13:44 98b5f35ea9d3eca6ed1881b5fe5d1e02024e1450822879e4c13bb48c9386d0ad.tar-split.gzroot 272894 Oct 19 13:44 98b5f35ea9d3eca6ed1881b5fe5d1e02024e1450822879e4c13bb48c9386d0ad.tar-split.gz
-rw-------. 1 root root    327 Oct 19 13:44 a3a542fc439141fb90369c89fdd23875fd4ec423983bd2631c2116b208776e6f.tar-split.gzroot    327 Oct 19 13:44 a3a542fc439141fb90369c89fdd23875fd4ec423983bd2631c2116b208776e6f.tar-split.gz
-rw-------. 1 root root  50969 Oct 19 13:44 c126300979bb0dfc017decea18ab61d3ea542febea9d4fe90f333e1cb3666db2.tar-split.gzroot  50969 Oct 19 13:44 c126300979bb0dfc017decea18ab61d3ea542febea9d4fe90f333e1cb3666db2.tar-split.gz
-rw-------. 1 root root  42672 Oct 19 13:44 c1a2d7288c9a4fc91cbfdc931f742cdb5aed45da08b2e6c780228856f30bb579.tar-split.gzroot  42672 Oct 19 13:44 c1a2d7288c9a4fc91cbfdc931f742cdb5aed45da08b2e6c780228856f30bb579.tar-split.gz
-rw-------. 1 root root     77 Oct 19 13:44 c1a9d7e93770930ea7a795125d0a2ba54bf7689d7c2b7dbbe7072971f1cd78fd.tar-split.gzroot     77 Oct 19 13:44 c1a9d7e93770930ea7a795125d0a2ba54bf7689d7c2b7dbbe7072971f1cd78fd.tar-split.gz
-rw-------. 1 root root   2729 Oct 19 13:44 layers.json   2729 Oct 19 13:44 layers.json
-rw-r--r--. 1 root root     64 Oct 19 13:44 layers.lock     64 Oct 19 13:44 layers.lock
-rw-------. 1 root root      2 Oct 10 09:43 volatile-layers.json      2 Oct 10 09:43 volatile-layers.json
```

As we can[]{#Chapter_5.xhtml#idx_6fd4433c .index-entry
index-entry="containers:storage driver"} see, we just found all the
layers\' archives downloaded []{#Chapter_5.xhtml#idx_e9b8059e
.index-entry index-entry="storage driver"}for our container image, but
where have they been unpacked? The answer is easy -- in the third
folder, `overlay`{.inlineCode}:

``` {.programlisting .snippet-con}
# ls -l overlay/
total 0
drwx------. 1 root root  46 Oct 19 13:44 7555495f10f71578f8bd7904214940724f63e98660a813f4fa6aa856dd22d8b7  46 Oct 19 13:44 7555495f10f71578f8bd7904214940724f63e98660a813f4fa6aa856dd22d8b7
drwx------. 1 root root  46 Oct 19 13:44 98b5f35ea9d3eca6ed1881b5fe5d1e02024e1450822879e4c13bb48c9386d0ad
drwx------. 1 root root  46 Oct 19 13:44 a3a542fc439141fb90369c89fdd23875fd4ec423983bd2631c2116b208776e6f  46 Oct 19 13:44 a3a542fc439141fb90369c89fdd23875fd4ec423983bd2631c2116b208776e6f
drwx------. 1 root root  46 Oct 19 13:44 c126300979bb0dfc017decea18ab61d3ea542febea9d4fe90f333e1cb3666db2  46 Oct 19 13:44 c126300979bb0dfc017decea18ab61d3ea542febea9d4fe90f333e1cb3666db2
drwx------. 1 root root  46 Oct 19 13:44 c1a2d7288c9a4fc91cbfdc931f742cdb5aed45da08b2e6c780228856f30bb579  46 Oct 19 13:44 c1a2d7288c9a4fc91cbfdc931f742cdb5aed45da08b2e6c780228856f30bb579
drwx------. 1 root root  46 Oct 19 13:44 c1a9d7e93770930ea7a795125d0a2ba54bf7689d7c2b7dbbe7072971f1cd78fd  46 Oct 19 13:44 c1a9d7e93770930ea7a795125d0a2ba54bf7689d7c2b7dbbe7072971f1cd78fd
drwxr-xr-x. 1 root root 312 Oct 19 13:44 lroot 312 Oct 19 13:44 l
```

The first []{#Chapter_5.xhtml#idx_7beeeb62 .index-entry
index-entry="containers:storage driver"}question that could arise when
looking at the latest directory[]{#Chapter_5.xhtml#idx_05e0a10c
.index-entry index-entry="storage driver"} content is, What\'s the
purpose of the `l`{.inlineCode} (*L* in lowercase) directory?

To answer this question, we have to inspect the content of a layer
directory.[]{.sentence-end} We can start with the first one on the list:

``` {.programlisting .snippet-con}
# ls -la overlay/7555495f10f71578f8bd7904214940724f63e98660a813f4fa6aa856dd22d8b7/
total 8
drwx------. 1 root root  46 Oct 19 13:44 .
drwx------. 1 root root 806 Oct 19 13:44 ..
dr-xr-xr-x. 1 root root   6 Oct 19 13:44 diff
-rw-r--r--. 1 root root  26 Oct 19 13:44 link
-rw-r--r--. 1 root root 144 Oct 19 13:44 lower
drwx------. 1 root root   0 Oct 19 13:44 merged   0 Oct 19 13:44 merged
drwx------. 1 root root   0 Oct 19 13:44 work   0 Oct 19 13:44 work
```

Let\'s understand the purpose of these files and directories:

-   `diff`{.inlineCode}: This directory represents the upper layer of
    the overlay, and is used to store any changes to the layer
-   `lower`{.inlineCode}: This file reports all the lower-layer mounts,
    ordered from uppermost to lowermost
-   `merged`{.inlineCode}: This directory is the one that the overlay is
    mounted on
-   `work`{.inlineCode}: This directory is used for internal operations
-   `link`{.inlineCode}: This file contains a unique string for the
    layer

Now, coming back to our question (What\'s the purpose of the
`l`{.inlineCode} (*L* in lowercase) directory?),

under the `l`{.inlineCode} directory, there are symbolic links with
unique strings pointing to the `diff`{.inlineCode} directory
[]{#Chapter_5.xhtml#idx_a82e49b7 .index-entry
index-entry="storage driver"}for []{#Chapter_5.xhtml#idx_af9ef77d
.index-entry index-entry="containers:storage driver"}every
layer.[]{.sentence-end} The symbolic links reference lower layers in the
`lower`{.inlineCode} file.[]{.sentence-end} Let\'s check it:

``` {.programlisting .snippet-con}
# ls -la overlay/l/
total 24
drwxr-xr-x. 1 root root 312 Oct 19 13:44 .
drwx------. 1 root root 806 Oct 19 13:44 ..
lrwxrwxrwx. 1 root root  72 Oct 19 13:44 3HBRITCBL5MXZGZHDNTU7LPC2X -> ../c126300979bb0dfc017decea18ab61d3ea542febea9d4fe90f333e1cb3666db2/diff
lrwxrwxrwx. 1 root root  72 Oct 19 13:44 43LDU3L34EOOEP5NFGNDBLNNNV -> ../98b5f35ea9d3eca6ed1881b5fe5d1e02024e1450822879e4c13bb48c9386d0ad/diff
lrwxrwxrwx. 1 root root  72 Oct 19 13:44 5FLKM24KJBZT2FVZI75PYPHEBW -> ../a3a542fc439141fb90369c89fdd23875fd4ec423983bd2631c2116b208776e6f/diff
lrwxrwxrwx. 1 root root  72 Oct 19 13:44 Q3ZVJUXUYIUWYRKBQYSD7R6OXT -> ../c1a2d7288c9a4fc91cbfdc931f742cdb5aed45da08b2e6c780228856f30bb579/diff
lrwxrwxrwx. 1 root root  72 Oct 19 13:44 RGJRWXLMSBUBJF4VR3RDLGFXP2 -> ../7555495f10f71578f8bd7904214940724f63e98660a813f4fa6aa856dd22d8b7/diff
lrwxrwxrwx. 1 root root  72 Oct 19 13:44 YJ4PBSR4UDAJMJVMWVK5P6WP2H -> ../c1a9d7e93770930ea7a795125d0a2ba54bf7689d7c2b7dbbe7072971f1cd78fd/diff
```

To double-check what we just learned, let\'s find the first layer of our
container image and check whether there is a `lower`{.inlineCode} file
for it.

Let\'s inspect the manifest file for our container image:

``` {.programlisting .snippet-con}
# cat overlay-images/1bcf11fa154f23987201bd92a75bf75e3507fc49f415d5dfe35887d1be3fd596/manifest | head -14
{
  "schemaVersion": 2,": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",": "application/vnd.oci.image.manifest.v1+json",
  "config": {
    "mediaType": "application/vnd.oci.image.config.v1+json",": "application/vnd.oci.image.config.v1+json",
    "digest": "sha256:1bcf11fa154f23987201bd92a75bf75e3507fc49f415d5dfe35887d1be3fd596",
    "size": 8015
  },
  "layers": [
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "digest": "sha256:a480a496ba95a197d587aa1d9e0f545ca7dbd40495a4715342228db62b67c4ba",
      "size": 29126289
    },
```

Then, we []{#Chapter_5.xhtml#idx_ff7cf661 .index-entry
index-entry="containers:storage driver"}must compare the checksum of the
compressed archive []{#Chapter_5.xhtml#idx_49dab50b .index-entry
index-entry="storage driver"}with the list of all the layers we
downloaded:

 note
**Good to** **know**

SHA-256 is an algorithm used to produce a unique cryptographic hash that
could be used to verify the integrity of a file
(checksum).[]{.sentence-end} Podman developers are working toward
support for other hash algorithms, such as SHA-512, which will offer
superior security properties.


``` {.programlisting .snippet-con}
# cat overlay-layers/layers.json | jq | grep -B3 -A10 "sha256:a480a"
  {
    "id": "98b5f35ea9d3eca6ed1881b5fe5d1e02024e1450822879e4c13bb48c9386d0ad",
    "created": "2024-10-19T13:44:05.710510598Z",
    "compressed-diff-digest": "sha256:a480a496ba95a197d587aa1d9e0f545ca7dbd40495a4715342228db62b67c4ba",
    "compressed-size": 29126289,
    "diff-digest": "sha256:98b5f35ea9d3eca6ed1881b5fe5d1e02024e1450822879e4c13bb48c9386d0ad",
    "diff-size": 77832192,
    "compression": 2,
    "uidset": [
      0
    ],
    "gidset": [
      0,
      8,
```

The file we just analyzed, `overlay-layers/layers.json`{.inlineCode},
was not indented.[]{.sentence-end} For this reason, we used the
`jq`{.inlineCode} utility to format it and make it human-readable.

 packt_tip
**Good to** **know**

If you cannot find the `jq`{.inlineCode} utility on your system, you can
install it through the operating system\'s default package
manager.[]{.sentence-end} On Fedora, for example, you can run
`dnf install`{.inlineCode}` `{.inlineCode}`jq`{.inlineCode}.


As you []{#Chapter_5.xhtml#idx_24ae952d .index-entry
index-entry="containers:storage driver"}can see, we just found the ID of
our root layer leveraging []{#Chapter_5.xhtml#idx_c512c206 .index-entry
index-entry="storage driver"}the `grep`{.inlineCode} utility searching
for the beginning of the hash string of our container image,
`"sha256:a480a"`{.inlineCode}.[]{.sentence-end} Now, let\'s look at its
content, inspecting the directory with the same ID that we found in the
previous command\'s output,
`98b5f35ea9d3eca6ed1881b5fe5d1e02024e1450822879e4c13bb48c9386d0ad`{.inlineCode}:

``` {.programlisting .snippet-con}
# ls -l overlay/98b5f35ea9d3eca6ed1881b5fe5d1e02024e1450822879e4c13bb48c9386d0ad
total 4
dr-xr-xr-x. 1 root root 132 Oct 19 13:44 diff
drwx------. 1 root root   0 Oct 19 13:44 empty
-rw-r--r--. 1 root root  26 Oct 19 13:44 link
drwx------. 1 root root   0 Oct 19 13:44 merged
drwx------. 1 root root   0 Oct 19 13:44 work
```

As we can verify, there is no `lower`{.inlineCode} file inside the
layer\'s directory because this is the first layer of our container
image!

The difference []{#Chapter_5.xhtml#idx_da089d04 .index-entry
index-entry="containers:storage driver"}we might notice is the presence
of a directory named `empty`{.inlineCode}.[]{.sentence-end}
This[]{#Chapter_5.xhtml#idx_969ca55d .index-entry
index-entry="storage driver"} is because if a layer has no parent, then
the overlay system will create a dummy lower directory named
`empty`{.inlineCode}, and it will skip writing a `lower`{.inlineCode}
file.

Finally, as the last stage of our practical example, let\'s run our
container and verify that a new `diff`{.inlineCode} layer will be
created.[]{.sentence-end} We expect that this layer will contain only
the difference between the lower ones.

 note
**Important** **note**

In the context of Podman\'s storage drivers (particularly the default
OverlayFS driver), a `diff`{.inlineCode} layer is the actual directory
on the host\'s disk that contains the specific filesystem changes
introduced by a single container or image layer.

While *layers* are a conceptual way to think about image building,
`diff`{.inlineCode} is the technical implementation of that layer in
your local storage.


First, we run the container image that we just analyzed:

``` {.programlisting .snippet-con}
# podman run -d docker.io/library/httpd:latest
9f84f1cbadf2c482dd2c4fa3ad350332bc956a4711e09decd9c66c1759f4f345
```

As you can see, we started it in the background through the
`-d`{.inlineCode} option to continue working on the system
host.[]{.sentence-end} After this, we will execute a new shell on the
pod to actually check the container\'s root folder and create a new file
in it:

``` {.programlisting .snippet-con}
# podman exec -ti 9f84f1cbadf2c482dd2c4fa3ad350332bc956a4711e09decd9c66c1759f4f345 /bin/bash
root@9f84f1cbadf2:/usr/local/apache2# pwd
/usr/local/apache2
root@9f84f1cbadf2:/usr/local/apache2# echo "this is my NOT persistent data" > tempfile.txt
root@9f84f1cbadf2:/usr/local/apache2# ls
bin  build  cgi-bin  conf  error  htdocs  icons  include  logs	modules  tempfile.txt
```

This new file []{#Chapter_5.xhtml#idx_70370c9b .index-entry
index-entry="containers:storage driver"}we just created will be
temporary and will only last for the lifetime
[]{#Chapter_5.xhtml#idx_9cdac8a3 .index-entry
index-entry="storage driver"}of the container.[]{.sentence-end} It is
now time to find the just-created `diff`{.inlineCode} layer created by
the overlay driver on our host system.[]{.sentence-end} The easiest way
is to analyze the mount points used in the running container:

``` {.programlisting .snippet-con}
root@9f84f1cbadf2:/usr/local/apache2# mount | grep overlay
overlay on / type overlay (rw,relatime,context="system_u:object_r:container_file_t:s0:c560,c898",lowerdir=/var/lib/containers/storage/overlay/l/RGJRWXLMSBUBJF4VR3RDLGFXP2:/var/lib/containers/storage/overlay/l/3HBRITCBL5MXZGZHDNTU7LPC2X:/var/lib/containers/storage/overlay/l/Q3ZVJUXUYIUWYRKBQYSD7R6OXT:/var/lib/containers/storage/overlay/l/YJ4PBSR4UDAJMJVMWVK5P6WP2H:/var/lib/containers/storage/overlay/l/5FLKM24KJBZT2FVZI75PYPHEBW:/var/lib/containers/storage/overlay/l/43LDU3L34EOOEP5NFGNDBLNNNV,upperdir=/var/lib/containers/storage/overlay/88633bde45bb8c7c25b5f42af4ba35d273fa7bf2c9fe689ae653675845dec9fc/diff,workdir=/var/lib/containers/storage/overlay/88633bde45bb8c7c25b5f42af4ba35d273fa7bf2c9fe689ae653675845dec9fc/work,redirect_dir=on,uuid=on,metacopy=on)
```

For rootless containers, the previous command will have to be run inside
the `podman unshare`{.inlineCode} shell.

As you can see, the first mount point of the list shows a very long line
full of layer paths divided by the colon.[]{.sentence-end} In this long
line, we can find the `upperdir`{.inlineCode} directory we are searching
for:

``` {.programlisting .snippet-con}
upperdir=/var/lib/containers/storage/overlay/88633bde45bb8c7c25b5f42af4ba35d273fa7bf2c9fe689ae653675845dec9fc/diff
```

Now, we can inspect the content of this directory and navigate through
the various available paths to find the container root directory where
we wrote that file in the previous commands:

``` {.programlisting .snippet-con}
# ls -la /var/lib/containers/storage/overlay/88633bde45bb8c7c25b5f42af4ba35d273fa7bf2c9fe689ae653675845dec9fc/diff/usr/local/apache2/
total 4
drwxr-xr-x. 1   33 tape 32 Oct 19 14:05 .
drwxr-xr-x. 1 root root 14 Oct 17 01:19 ..
drwxr-xr-x. 1 root root 18 Oct 19 14:04 logs
-rw-r--r--. 1 root root 31 Oct 19 14:05 tempfile.txt
# cat /var/lib/containers/storage/overlay/88633bde45bb8c7c25b5f42af4ba35d273fa7bf2c9fe689ae653675845dec9fc/diff/usr/local/apache2/tempfile.txt
this is my NOT persistent data
```

As we verified, the[]{#Chapter_5.xhtml#idx_f0328ac2 .index-entry
index-entry="containers:storage driver"} data is stored on the host
operating system, but it is []{#Chapter_5.xhtml#idx_2aa8f570
.index-entry index-entry="storage driver"}stored in a temporary layer
that will sooner or later be removed once the container is removed!

## The storage.conf configuration file {#Chapter_5.xhtml#h2_139 .heading-2}

Now, coming[]{#Chapter_5.xhtml#idx_f98cb19f .index-entry
index-entry="containers:storage.conf configuration file"} back to the
original topic that sent us on[]{#Chapter_5.xhtml#idx_39703701
.index-entry index-entry="storage.conf configuration file"} this small
trip under the hood of the overlay storage driver, we were talking about
`/etc/containers/storage.conf`{.inlineCode}.[]{.sentence-end} This file
holds all the configurations for the `containers/storage`{.inlineCode}
project that is responsible for sharing an underlying common method to
access container storage on a host.[]{.sentence-end} The other options
available in this file are related to the customization of the storage
driver, as well as changing the default paths for the internal storage
directories.[]{.sentence-end} Chief among these is
`graphroot`{.inlineCode} (often referred to as the root
path).[]{.sentence-end} This is the persistent storage location where
Podman saves all downloaded image layers, the local image metadata, and
the `diff`{.inlineCode} layers we discussed earlier.

Following this, the `runroot`{.inlineCode} directory serves a different
purpose.[]{.sentence-end} While `graphroot`{.inlineCode} handles
long-term storage, `runroot`{.inlineCode} is used to store temporary,
volatile data produced by active containers.[]{.sentence-end} This
includes state files, lock files, and the *merged* mount points that
exist only while a container is running.[]{.sentence-end} Because this
data is temporary and often high-performance, it is frequently stored in
a memory-backed filesystem (such as `tmpfs`{.inlineCode}) to ensure
speed and to ensure that it is cleared upon a system reboot.

If we inspect the folder on our running host where we started the
container for the previous example, we []{#Chapter_5.xhtml#idx_2153da41
.index-entry index-entry="storage.conf configuration file"}will find
that this is a general internal storage area for container files,
including PID files and internal log files, as well as certain files
that are automatically generated by Podman and are mounted into the
container:

``` {.programlisting .snippet-con}
# ls -l /run/containers/storage/overlay-containers/9f84f1cbadf2c482dd2c4fa3ad350332bc956a4711e09decd9c66c1759f4f345/userdata/
total 20
-rw-r--r--. 1 root root   4 Oct 19 14:04 conmon.pid
-rw-r--r--. 1 root root  13 Oct 19 14:04 hostname
-rw-r--r--. 1 root root 243 Oct 19 14:04 hosts
-rw-r--r--. 1 root root   0 Oct 19 14:04 oci-log
-rwx------. 1 root root   4 Oct 19 14:04 pidfile
-rw-r--r--. 1 root root  25 Oct 19 14:04 resolv.conf
drwxr-xr-x. 3 root root  60 Oct 19 14:04 run
```

As you can see []{#Chapter_5.xhtml#idx_a2e3deb8 .index-entry
index-entry="containers:storage.conf configuration file"}from the
preceding output, the container\'s folder under the
`runroot`{.inlineCode} path contains various files that have been
mounted directly onto the container to customize it.

To wrap up, in the previous examples, we analyzed the anatomy of a
container image and what happens once we run a new container from that
image.[]{.sentence-end} The technology behind the scenes is amazing, and
we saw that a lot of features are related to the isolation capabilities
offered by the operating system.[]{.sentence-end} Here, storage offers
other important functionalities that have made containers the greatest
technology that we all now know about.[]{.sentence-end} Closely tied to
container storage is the ability to access local or remote files from
within a container.[]{.sentence-end} In the following section, we will
explore several techniques for managing this data exchange effectively.

# Copying files in and out of a container {#Chapter_5.xhtml#h1_140 .heading-1}

Podman enables[]{#Chapter_5.xhtml#idx_179eaa1f .index-entry
index-entry="containers:files, copying in and out"} users to move files
in and out of a running container.[]{.sentence-end} There are two main
methods for achieving this goal:

-   Using the `podman cp`{.inlineCode} command
-   Mounting the container\'s filesystem

Let\'s see these two techniques in more depth with some examples.

## Using podman cp {#Chapter_5.xhtml#h2_141 .heading-2}

This result is []{#Chapter_5.xhtml#idx_ed3f0dca .index-entry
index-entry="files, copying in and out of container:podman cp, using"}achieved
using the `podman cp`{.inlineCode} command, which can move files and
folders to and from a container.[]{.sentence-end} Its usage is quite
simple and will be illustrated in the next example.

First, let\'s start a new Alpine container:

``` {.programlisting .snippet-con}
$ podman run -d --name alpine_cp_test alpine sleep 1000
```

Now, let\'s grab a file from the container -- we have chosen the
`/etc/os-release`{.inlineCode} file, which provides some information
about the distribution and its version ID:

``` {.programlisting .snippet-con}
$ podman cp alpine_cp_test:/etc/os-release /tmp
```

The file has been copied to the host `/tmp`{.inlineCode} folder and can
be inspected:

``` {.programlisting .snippet-con}
$ cat /tmp/os-release
NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.20.3
PRETTY_NAME="Alpine Linux v3.20"
HOME_URL=https://alpinelinux.org/
BUG_REPORT_URL="https://gitlab.alpinelinux.org/alpine/aports/-/issues"
```

In the opposite direction, we can copy files or folders from the host to
the running container or from a container into another:

``` {.programlisting .snippet-con}
$ podman cp /tmp/build_folder alpine_cp_test:/
```

This example copies the `/tmp/build_folder`{.inlineCode} folder, and all
its content, under the root filesystem of
[]{#Chapter_5.xhtml#idx_f6f7f3e1 .index-entry
index-entry="files, copying in and out of container:podman cp, using"}the
Alpine container.[]{.sentence-end} We can then inspect the result of the
`copy`{.inlineCode} command by using `podman exec`{.inlineCode} with the
`ls`{.inlineCode} utility command.

## Interacting with overlayfs {#Chapter_5.xhtml#h2_142 .heading-2}

There is []{#Chapter_5.xhtml#idx_0f29d8d5 .index-entry
index-entry="files, copying in and out of container:overlayfs, interacting with"}another
way to copy files from a container to the host, which is by using the
`podman mount`{.inlineCode} command and interacting directly with the
merged overlays.

To mount a running rootless container\'s filesystem, we first need to
run the `podman unshare`{.inlineCode} command, which permits users to
run commands inside a modified user namespace:

``` {.programlisting .snippet-con}
$ podman unshare
```

This command drops a root shell in a new user namespace configured with
*UID 0* and *GID 0*.[]{.sentence-end} It is now possible to run the
`podman mount`{.inlineCode} command and obtain the absolute path of the
mount point:

``` {.programlisting .snippet-con}
# cd $(podman mount alpine_cp_test)
```

The preceding command uses shell expansion to change to the path of
`MergedDir`{.inlineCode}, which, as the name suggests, merges the
`LowerDir`{.inlineCode} and `UpperDir`{.inlineCode} contents to provide
a unified view of the different layers.[]{.sentence-end} From now on, it
is possible to copy files to and from the container root
filesystem.[]{.sentence-end} This is a great way to quickly dig into the
mounted filesystem of a container.

The previous examples were based on rootless containers, but the same
logic applies to rootful containers.[]{.sentence-end} The practice of
copying files and folders from a container is especially useful for
troubleshooting purposes.[]{.sentence-end} The opposite action of
copying them inside a running container can be useful for updating and
testing secrets or configuration files.[]{.sentence-end} In that case,
we have the option of persisting those changes, as described in the next
subsection.

### Persisting changes with podman commit {#Chapter_5.xhtml#h3_143 .heading-3}

The previous []{#Chapter_5.xhtml#idx_9ae38b5f .index-entry
index-entry="files, copying in and out of container:changes, persisting with podman commit"}examples
are not a method for permanently customizing running containers, since
the immutable nature of containers implies that persistent modifications
should go through an image rebuild.

However, if we need to preserve the changes and produce a new image
without starting a new build, the `podman commit`{.inlineCode} command
provides a way to persist the changes to a container into a new image.

The commit concept is of primary importance in Docker and OCI image
builds.[]{.sentence-end} In fact, we can interpret the different steps
of a Dockerfile as a series of commits applied during the build process.

The following example shows how to persist a file copied into a running
container and produce a new image.[]{.sentence-end} Let\'s say we want
to update the default `index.html`{.inlineCode} page of our
`n`{.inlineCode}`ginx`{.inlineCode} container:

``` {.programlisting .snippet-con}
$ echo "Hello World!" > /tmp/index.html
$ podman run --name custom_nginx -d -p \
  8080:80 docker.io/library/nginx
$ podman cp /tmp/index.html \
  custom_nginx:/usr/share/nginx/html/
```

Let\'s test the changes applied:

``` {.programlisting .snippet-con}
$ curl localhost:8080
Hello World!
```

Now, we want to persist the changed `index.html`{.inlineCode} file into
a new image, starting from the running container with
`podman commit`{.inlineCode}:

``` {.programlisting .snippet-con}
$ podman commit -p custom_nginx hello-world-nginx
```

The preceding command persists the changes by effectively creating a new
image layer containing the updated files and folders.[]{.sentence-end}
All layers except the newest one, the one just created, are shared with
the existing `nginx`{.inlineCode} image.[]{.sentence-end} No extra
storage is used except for the new layer.

The []{#Chapter_5.xhtml#idx_735eef60 .index-entry
index-entry="files, copying in and out of container:changes, persisting with podman commit"}previous
container can now be safely stopped and removed before testing the new
custom image:

``` {.programlisting .snippet-con}
$ podman stop custom_nginx && podman rm custom_nginx
```

Let\'s test the new custom image and inspect the changed
`index.html`{.inlineCode} file:

``` {.programlisting .snippet-con}
$ podman run -d -p 8080:80 --name hello_world \
  localhost/hello-world-nginx
$ curl localhost:8080
Hello World!
```

In this section, we have learned how to copy files to and from a running
container and how to commit the changes on the fly by producing a new
image.

In the next section, we are going to learn how host storage is attached
to a container by introducing the concept of **volumes** and **bind
mounts**.

# Attaching host storage to a container {#Chapter_5.xhtml#h1_144 .heading-1}

We have already[]{#Chapter_5.xhtml#idx_5350e24e .index-entry
index-entry="containers:host storage, attaching"} talked about the
immutable nature of containers.[]{.sentence-end}
Starting[]{#Chapter_5.xhtml#idx_79997ca9 .index-entry
index-entry="host storage"} from prebuilt images, when we run a
container, we instantiate a read/write layer on top of a stack of
read-only layers using a COW approach.

Containers []{#Chapter_5.xhtml#idx_306064f6 .index-entry
index-entry="host storage:attaching to container"}are ephemeral objects
based on a stateful image.[]{.sentence-end} This implies that containers
are not meant to store data inside them.[]{.sentence-end} If a container
crashes or is removed, all the data would be lost.[]{.sentence-end} We
need a way to store data in a separate location that is mounted inside
the running container, preserved when the container is removed, and
ready to be reused by a new container.

There is another important caveat that should not be forgotten:
**secrets** and **config files**.[]{.sentence-end} When we build an
image, we can pass all the files and folders we need inside
it.[]{.sentence-end} However, sealing secrets such as certificates or
keys inside a build is not a good practice.[]{.sentence-end} If we need,
for example, to rotate a certificate, we\'ll have to rebuild the whole
image from scratch.[]{.sentence-end} In the same way, changing
a[]{#Chapter_5.xhtml#idx_036405ed .index-entry
index-entry="host storage:attaching to container"} config file that
resides inside an image[]{#Chapter_5.xhtml#idx_c1174715 .index-entry
index-entry="containers:host storage, attaching"} implies a new rebuild
every time we change a setting.

For these reasons, OCI specifications support volumes and bind mounts to
manage storage attached to a container.[]{.sentence-end} In the next
sections, we will learn how volumes and bind mounts work and how to
attach them to a container.

## Managing and attaching bind mounts to a container {#Chapter_5.xhtml#h2_145 .heading-2}

Let\'s start with []{#Chapter_5.xhtml#idx_c3fe2a62 .index-entry
index-entry="host storage:bind mounts, managing and attaching to container"}bind
mounts since they leverage a native Linux feature.[]{.sentence-end}
According to the official Linux man pages, a bind mount is *a way to
remount a part of the filesystem hierarchy somewhere
else*.[]{.sentence-end} This means that
using[]{#Chapter_5.xhtml#idx_92ed6a13 .index-entry
index-entry="bind mounts"} bind mounts, we can replicate the view of a
directory under another mount point in the host.

Before []{#Chapter_5.xhtml#idx_87d6ec79 .index-entry
index-entry="bind mounts:managing and attaching, to container"}learning
how containers use bind mounts, let\'s see a basic example where we
simply bind-mount the `/etc`{.inlineCode} directory under the
`/mnt`{.inlineCode} directory:

``` {.programlisting .snippet-con}
$ sudo mount --bind /etc /mnt
```

After issuing this command, we will see the exact contents of
`/etc`{.inlineCode} under `/mnt`{.inlineCode}.[]{.sentence-end} To
unmount, simply run the following command:

``` {.programlisting .snippet-con}
$ sudo umount /mnt
```

The same concept can be applied to containers.[]{.sentence-end} Podman
can bind-mount host directories inside a container and offers dedicated
CLI options to simplify the mount process.

Podman offers two options that can be used to bind-mount:
`-v|--volume`{.inlineCode} and `–mount`{.inlineCode}.[]{.sentence-end}
Let\'s cover these in more detail.

### The -v\|\--volume option {#Chapter_5.xhtml#h3_146 .heading-3}

This option []{#Chapter_5.xhtml#idx_6bbf06ad .index-entry
index-entry="bind mounts:-v|–volume option"}uses a compact, single-field
argument to define the source host directory and the container mount
point with the `/HOST_DIR:/CONTAINER_DIR`{.inlineCode}
pattern.[]{.sentence-end} The following example mounts the
`/host_files`{.inlineCode} directory on the `/mnt`{.inlineCode} mount
point inside the container:

``` {.programlisting .snippet-con}
$ podman run -v /host_files:/mnt docker.io/library/nginx
```

It is possible to pass extra arguments to define mount behavior, for
example, to mount the host directory as read-only:

``` {.programlisting .snippet-con}
$ podman run –v /host_files:/mnt:ro \
   docker.io/library/nginx
```

Other viable options for bind mounts using the
`-v|--volume`{.inlineCode} option can be found in the `run`{.inlineCode}
command man page (`man podman-run`{.inlineCode}).

### The \--mount option {#Chapter_5.xhtml#h3_147 .heading-3}

This []{#Chapter_5.xhtml#idx_a81d6d3a .index-entry
index-entry="bind mounts:–mount option"}option is more verbose since it
uses a *key=value* syntax to define the source and destinations as well
as the mount type and extra arguments.[]{.sentence-end} This option
accepts different mount types (`bind`{.inlineCode} (bind mount),
`volume`{.inlineCode}, `tmpfs`{.inlineCode}, `image`{.inlineCode}, and
`devpts`{.inlineCode}) in the
`type=TYPE`{.inlineCode}`,source`{.inlineCode}`=HOST_DIR,destination=CONTAINER_DIR`{.inlineCode}
pattern.[]{.sentence-end} The `source`{.inlineCode} and
`destination`{.inlineCode} keys can be replaced with the shorter
`src`{.inlineCode} and `dst`{.inlineCode},
respectively.[]{.sentence-end} The previous example can be rewritten as
follows:

``` {.programlisting .snippet-con}
$ podman run \
  --mount type=bind,src=/host_files,dst=/mnt \
  docker.io/library/nginx
```

 packt_tip
**Important** **note**

When using bind mounts on systems with SELinux enabled (such as Fedora,
RHEL, or CentOS), the container may be denied access to the host files
due to security labels.[]{.sentence-end} To resolve this, Podman allows
you to append a relabeling option to the mount:

-   `type=`{.inlineCode}`bind,src=...,dst=...,relabel=shared`{.inlineCode}
    (or `:z`{.inlineCode}): This tells Podman that the content will be
    shared between multiple containers.[]{.sentence-end} It applies a
    shared SELinux security label to the host files.
-   `type=`{.inlineCode}`bind,src=...,dst=...,relabel=private`{.inlineCode}
    (or `:Z`{.inlineCode}): This tells Podman that the content is
    private and unshared.[]{.sentence-end} It applies a unique,
    exclusive security label to the host files, ensuring only that
    specific container can access them.

**Warning**: Be []{#Chapter_5.xhtml#idx_cb175bb8 .index-entry
index-entry="bind mounts:–mount option"}cautious when using these flags
on system-critical directories (such as `/etc`{.inlineCode} or
`/home`{.inlineCode}), as relabeling these files can prevent other host
processes or the operating system from accessing them correctly.


We can also pass an extra option by adding an extra comma, for example,
to mount the host directory as read-only:

``` {.programlisting .snippet-con}
$ podman run \
  --mount type=bind,src=/host_files,dst=/mnt,ro=true \
  docker.io/library/nginx
```

Despite being very simple to use and understand, bind mounts have some
limitations that could impact the life cycle of the container in some
cases.[]{.sentence-end} Host files and directories must exist before
running the containers, and permissions must be set accordingly to make
them readable or writable.[]{.sentence-end} Another important caveat to
keep in mind is that a bind mount always obfuscates the underlying mount
point in the container if populated by files or
directories.[]{.sentence-end} A useful alternative to bind mounts is
volumes, described in the next subsection.

## Managing and attaching volumes to a container {#Chapter_5.xhtml#h2_148 .heading-2}

A volume []{#Chapter_5.xhtml#idx_64c6a403 .index-entry
index-entry="volume"}is a []{#Chapter_5.xhtml#idx_9dcad3e4 .index-entry
index-entry="host storage:volumes, managing and attaching to container"}directory
[]{#Chapter_5.xhtml#idx_910c4f2d .index-entry
index-entry="volume:managing and attaching, to container"}created and
managed directly by the container engine and mounted to a mount point
inside the container.[]{.sentence-end} They offer a great solution for
persisting data generated by a container; instead of managing the data
and directory yourself, Podman does it for you.

Volumes []{#Chapter_5.xhtml#idx_cbf1aee0 .index-entry
index-entry="volume:managing and attaching, to container"}can
[]{#Chapter_5.xhtml#idx_552efb4d .index-entry index-entry="volume"}be
managed using the `podman volume`{.inlineCode} command, which can be
used to list, inspect, create, and remove volumes in the
system.[]{.sentence-end} Let\'s start with a basic example, with a
volume automatically created by Podman on top of the
`n`{.inlineCode}`ginx`{.inlineCode} document root:

``` {.programlisting .snippet-con}
$ podman run -d -p 8080:80  --name nginx_volume1 -v /usr/share/nginx/html docker.io/library/nginx
```

This time, the `–v`{.inlineCode} option has an argument with only one
item -- the document root directory.[]{.sentence-end} In this case,
Podman automatically creates a volume and bind-mounts it to the target
mount point.

To prove that []{#Chapter_5.xhtml#idx_d89320ce .index-entry
index-entry="host storage:volumes, managing and attaching to container"}a
new volume has been created, we can inspect the container:

``` {.programlisting .snippet-con}
$ podman inspect nginx_volume1
[...omitted output...]
"Mounts": [
          {
                "Type": "volume",
                "Name": "2ed93716b7ad73706df5c6f56bda262920accec59e7b6642d36f938e936d36d9",
                "Source": "/home/packt/.local/share/containers/storage/volumes/2ed93716b7ad73706df5c6f56bda262920accec59e7b6642d36f938e936d36d9/_data",
                "Destination": "/usr/share/nginx/html",
                "Driver": "local",
                "Mode": "",
                "Options": [
                    "nosuid",
                    "nodev",
                    "rbind"
                ],
                "RW": true,
                "Propagation": "rprivate"
            }
        ],
[…omitted output]
```

In the `Mounts`{.inlineCode} section, we have a list of objects mounted
in the container.[]{.sentence-end} The only item is an object of the
`volume`{.inlineCode} type, with a generated UID as its name and a
`Source`{.inlineCode} field that represents its path in
[]{#Chapter_5.xhtml#idx_8a9eec6e .index-entry
index-entry="volume:managing and attaching, to container"}the host,
while the `Destination`{.inlineCode} field is the mount point inside the
container.

We can []{#Chapter_5.xhtml#idx_493f041a .index-entry
index-entry="host storage:volumes, managing and attaching to container"}double-check
the existence of the volume with the `podman volume ls`{.inlineCode}
command:

``` {.programlisting .snippet-con}
$ podman volume ls
DRIVER VOLUME NAME
local  2ed93716b7ad73706df5c6f56bda262920accec59e7b6642d36f938e936d36d9
```

We can inspect the local folder by inspecting its location and then
looking inside the source path.[]{.sentence-end} As we can see, we will
find the default files in the container document root:

``` {.programlisting .snippet-con}
$ podman volume inspect $ID | grep Mountpoint
$ ls -al
/home/packt/.local/share/containers/storage/volumes/2ed93716b7ad73706df5c6f56bda262920accec59e7b6642d36f938e936d36d9/_data
total 16
drwxr-xr-x. 2 packt packt 4096 Sep  9 20:26 .
drwx------. 3 packt packt 4096 Oct 16 22:41 ..
-rw-r--r--. 1 packt packt  497 Sep  7 17:21 50x.html
-rw-r--r--. 1 packt packt  615 Sep  7 17:21 index.html
```

This demonstrated that when an empty volume is created, it is populated
with the content of the target mount point.[]{.sentence-end} When a
container stops, the volume is preserved along with all the data and can
be reused when the container is restarted by another
container.[]{.sentence-end} Entering the containers world, you will
notice that popular container images are already using volumes by
default for storing their configuration and data in a structured way.

The preceding example shows a volume with a generated UID, but it is
possible to choose the name of the attached volume, as in the following
example:

``` {.programlisting .snippet-con}
$ podman run -d -p 8080:80  --name nginx_volume2 -v nginx_vol:/usr/share/nginx/html docker.io/library/nginx
```

In the []{#Chapter_5.xhtml#idx_458fcb88 .index-entry
index-entry="volume:managing and attaching, to container"}preceding
example, Podman creates a new volume named `nginx_vol`{.inlineCode} and
stores it under the default `volumes`{.inlineCode}
directory.[]{.sentence-end} When a named volume is created, Podman does
not need to generate a UID.

The default `volumes`{.inlineCode} directory has different paths for
rootless and rootful containers:

-   For rootless containers, the default volume storage path is
    `<`{.inlineCode}`USER_HOME`{.inlineCode}`>`{.inlineCode}`/.local/share/containers/storage/volumes`{.inlineCode}
-   For rootful containers, the default volume storage path is
    `/var/lib/containers/storage/volumes`{.inlineCode}

Alternatively, Podman offers `podman volume mount`{.inlineCode} and
`podman volume unmount`{.inlineCode} commands to easily get access to
the files of a single volume.

Volumes []{#Chapter_5.xhtml#idx_e5025ab9 .index-entry
index-entry="host storage:volumes, managing and attaching to container"}created
in those paths are persisted after the container is destroyed and can be
reused by other containers.

To manually remove a volume, use the `podman volume rm`{.inlineCode}
command if no containers are using that volume:

``` {.programlisting .snippet-con}
$ podman volume rm nginx_vol
```

In case of containers using a certain volume, you could instead use the
following command, which removes the volume and any containers using it:

``` {.programlisting .snippet-con}
$ podman volume rm --force nginx_vol
```

When dealing with multiple volumes, the
`podman volume prune`{.inlineCode} command removes all the unused
volumes.[]{.sentence-end} The following example prunes all the volumes
in the user default volume storage (the one used by rootless
containers):

``` {.programlisting .snippet-con}
$ podman volume prune
```

The same commands work for root and rootless.

 packt_tip
**Important** **note**

Do not forget to monitor volumes accumulating in the host since they
consume disk space that could be reclaimed, and prune unused volumes
periodically to avoid cluttering the host storage.[]{.sentence-end}
Additionally, containers based on certain images can automatically
create volumes that are not always removed when the container is
removed, leading to unexpected accumulation of volumes.


Users can []{#Chapter_5.xhtml#idx_914ed8ae .index-entry
index-entry="host storage:volumes, managing and attaching to container"}also
[]{#Chapter_5.xhtml#idx_00f293ea .index-entry
index-entry="volume:managing and attaching, to container"}preliminarily
create and populate volumes before running the
container.[]{.sentence-end} The following example uses the
`podman create volume`{.inlineCode} command to create the volume mounted
to the `n`{.inlineCode}`ginx`{.inlineCode} document root and then
populates it with a test `index.html`{.inlineCode} file:

``` {.programlisting .snippet-con}
$ podman volume create custom_nginx
$ podman unshare
# podman volume mount custom_nginx
/home/packt/.local/share/containers/storage/volumes/custom_nginx/_data
# echo "Hello World!" >>  /home/packt/.local/share/containers/storage/volumes/custom_nginx/_data/index.html
```

We can now run a new `n`{.inlineCode}`ginx`{.inlineCode} container using
the prepopulated volume:

``` {.programlisting .snippet-con}
$ podman run -d -p 8080:80  --name nginx_volume3 -v custom_nginx:/usr/share/nginx/html docker.io/library/nginx
```

The HTTP test shows the updated contents:

``` {.programlisting .snippet-con}
$ curl localhost:8080
Hello World!
```

This time, the []{#Chapter_5.xhtml#idx_f46a034d .index-entry
index-entry="host storage:volumes, managing and attaching to container"}volume,
which was []{#Chapter_5.xhtml#idx_69ced0c0 .index-entry
index-entry="volume:managing and attaching, to container"}not empty in
the beginning, obfuscated the container target directory with its
contents.

As with bind mounts, we can freely choose between the
`-v|--volume`{.inlineCode} and the `--mount`{.inlineCode}
options.[]{.sentence-end} Let\'s see how.

### Mounting volumes with the \--mount option {#Chapter_5.xhtml#h3_149 .heading-3}

The following []{#Chapter_5.xhtml#idx_cbaef915 .index-entry
index-entry="volume:mounting, with –mount option"}example runs an
`n`{.inlineCode}`ginx`{.inlineCode} container using the
`--mount`{.inlineCode} flag:

``` {.programlisting .snippet-con}
$ podman run -d -p 8080:80  --name nginx_volume4 --mount type=volume,src=custom_nginx,dst=/usr/share/nginx/html docker.io/library/nginx
```

While the `-v|--volume`{.inlineCode} option is compact and widely
adopted, the advantage of the `--mount`{.inlineCode} option is a clearer
and more expressive syntax, along with an exact statement of the mount
type; however, the `-v|--volume`{.inlineCode} option is statistically
more used because of its simpler syntax.

### Volume drivers {#Chapter_5.xhtml#h3_150 .heading-3}

The preceding []{#Chapter_5.xhtml#idx_df8bf784 .index-entry
index-entry="volume:volume drivers"}volume examples are all based on the
same **local** volume driver, which is used to manage volume in the
local filesystem of the host.[]{.sentence-end} Additional volume drivers
can be configured in the
`/usr/share/containers/containers.conf`{.inlineCode} file in the
`[engine.volume_plugins]`{.inlineCode} section by passing the plugin
name followed by the file or socket path.[]{.sentence-end} Certain
vendors provide volume plugins that can be used with this system, but it
is not usually necessary unless you have a storage solution offering
such a plugin.

The local volume driver can also be used to mount **NFS** shares in the
host running the container.[]{.sentence-end} The following example shows
how to create a volume backed by an NFS share and mount it inside a
MongoDB container on its `/data/db`{.inlineCode} directory:

``` {.programlisting .snippet-con}
$ sudo podman volume create --driver local --opt type=nfs --opt o=addr=nfs-host.example.com,rw,context="system_u:object_r:container_file_t:s0" --opt device=:/opt/nfs-export nfs-volume
$ sudo podman run -d -v nfs-volume:/data/db docker.io/library/mongo
```

A prerequisite []{#Chapter_5.xhtml#idx_5750c9c0 .index-entry
index-entry="volume:volume drivers"}of the preceding example is the
preliminary configuration of the NFS server, which should be accessible
by the host running the container.[]{.sentence-end} It is important to
notice that this feature is not available for rootless containers.

### Volumes in builds {#Chapter_5.xhtml#h3_151 .heading-3}

Volumes[]{#Chapter_5.xhtml#idx_ee6ec43b .index-entry
index-entry="volume:in builds"} can be predefined during the image build
process.[]{.sentence-end} This lets image maintainers define which
container directories will be automatically attached to
volumes.[]{.sentence-end} To understand this concept, let\'s inspect
this minimal Dockerfile:

``` {.programlisting .snippet-code}
FROM docker.io/library/nginx:latest
VOLUME /usr/share/nginx/html
```

The only change made to the `docker.io/library/nginx`{.inlineCode} image
is a `VOLUME`{.inlineCode} directive, which defines which directory
should be externally mounted as an anonymous volume in the
host.[]{.sentence-end} This declaration is simply metadata, and the
volume will be created only at runtime when a container is started from
this image.

If we build the image and run a container based on the example
Dockerfile, we can see an automatically created anonymous volume:

``` {.programlisting .snippet-con}
$ podman build -t my_nginx .
$ podman run -d --name volumes_from_build my_nginx
$ podman inspect volumes_from_build --format "{{ .Mounts }}"
[{volume 4d6ac7edcb4f01add205523b7733d61ae4a5772786eacca68e4972b20fd1180c /home/packt/.local/share/containers/storage/volumes/4d6ac7edcb4f01add205523b7733d61ae4a5772786eacca68e4972b20fd1180c/_data /usr/share/nginx/html local  [nodev exec nosuid rbind] true rprivate}]
```

Without an explicit volume creation option, Podman has already created
and mounted the container volume.[]{.sentence-end} This automatic volume
definition at build time is a common practice in all containers that
[]{#Chapter_5.xhtml#idx_c944e1bc .index-entry
index-entry="volume:in builds"}are expected to persist data, such as
databases.[]{.sentence-end} Please note that in most cases, volumes
created with the container and not given a name, like this one, are
automatically removed with the container that created them.

For example, let\'s inspect the official MongoDB image: the
`docker.io/library/mongo`{.inlineCode} image is already configured to
create two volumes, one for `/data/configdb`{.inlineCode} and one for
`/data/db`{.inlineCode}.[]{.sentence-end} The same behavior can be
identified in the most common databases, including PostgreSQL, MariaDB,
or MySQL.

It is possible to define how predefined anonymous volumes should be
mounted when a container starts.[]{.sentence-end} By default, volumes
are created and bind-mounted into the container (referred to as the
**bind** option).[]{.sentence-end} However, you can also choose to mount
them as `tmpfs`{.inlineCode} or ignore them entirely using the
`--image-volume`{.inlineCode} option.[]{.sentence-end} The following
example starts a MongoDB container with its default volumes mounted as
`tmpfs`{.inlineCode}:

``` {.programlisting .snippet-con}
$ podman run -d --image-volume tmpfs docker.io/library/mongo
```

In *[Chapter 6](#Chapter_6.xhtml#h1_163){.chapref}*, we will cover the
build process in greater detail.[]{.sentence-end} We now close this
subsection with an example of how to mount volumes across multiple
containers.

### Mounting volumes across containers {#Chapter_5.xhtml#h3_152 .heading-3}

One of the greatest []{#Chapter_5.xhtml#idx_06eea6d0 .index-entry
index-entry="volume:mounting, across containers"}advantages of volumes
is their flexibility.[]{.sentence-end} For example, a container can
mount volumes from an already running container to share the same
data.[]{.sentence-end} To accomplish this result, we can use the
`--volumes-from`{.inlineCode} option.[]{.sentence-end} The following
example starts a MongoDB container and then cross-mounts its volumes on
a Fedora container:

``` {.programlisting .snippet-con}
$ podman run -d --name mongodb01 docker.io/library/mongo
$ podman run -it --volumes-from=mongodb01 docker.io/library/fedora
```

The second container drops an interactive root shell we can use to
inspect the filesystem content:

``` {.programlisting .snippet-con}
[root@c10420016687 /]# ls -al /data
total 20
drwxr-xr-t.  4 root root 4096 Oct 17 15:36 .
dr-xr-xr-x. 19 root root 4096 Oct 17 15:36 ..
drwxr-xr-x.  2  999  999 4096 Sep 20 22:20 configdb
drwxr-xr-x.  4  999  999 4096 Oct 17 15:36 db
```

As expected, we []{#Chapter_5.xhtml#idx_223bd102 .index-entry
index-entry="volume:mounting, across containers"}can find the MongoDB
volumes mounted in the Fedora container.[]{.sentence-end} If we stop and
even remove the `mongodb01`{.inlineCode} container, the volumes remain
active and mounted inside the Fedora container.[]{.sentence-end} We can
easily achieve the same result using the `podman inspect`{.inlineCode}
command.

By default, Podman mounts the volumes with the same permissions as they
are mounted in the source container.[]{.sentence-end} It is possible to
change them by adding the comma-separated `rw`{.inlineCode} or
`ro`{.inlineCode} option.[]{.sentence-end} The following example
attaches the volume from the MongoDB container as a read-only volume:

``` {.programlisting .snippet-con}
$ podman run -it --volumes-from=mongodb01,ro docker.io/library/fedora
```

Until now, we have seen basic use cases with no specific segregation
between containers or mounted resources.[]{.sentence-end} If the host
has SELinux enabled and in enforcing mode, some extra considerations
must be applied.

## SELinux considerations for mounts {#Chapter_5.xhtml#h2_153 .heading-2}

SELinux recursively []{#Chapter_5.xhtml#idx_440c3273 .index-entry
index-entry="host storage:SELinux considerations, for mounts"}applies
labels to files and directories to define their
context.[]{.sentence-end} Those labels are usually stored as extended
filesystem []{#Chapter_5.xhtml#idx_c8db6a1e .index-entry
index-entry="SELinux:considerations for mounts"}attributes.[]{.sentence-end}
SELinux uses contexts to manage policies and define which processes can
access a specific resource.

The `ls`{.inlineCode} command is used to see the type context of a
resource:

``` {.programlisting .snippet-con}
$ ls -alZ /etc/passwd
-rw-r--r--. 1 root root system_u:object_r:passwd_file_t:s0 2965 Jul 28 21:00 /etc/passwd
```

In the preceding example, the `passwd_file_t`{.inlineCode} label defines
the type context of the `/etc/passwd`{.inlineCode}
file.[]{.sentence-end} Depending on the type context, a program can or
cannot access a file while SELinux is running in enforcing mode.

Processes also []{#Chapter_5.xhtml#idx_cc521810 .index-entry
index-entry="host storage:SELinux considerations, for mounts"}have their
type context -- containers run with the label `container_t`{.inlineCode}
and have read/write access to files and directories labeled with
`container_file_t`{.inlineCode} type context, and read/execute access to
`container_share_t`{.inlineCode} labeled resources.

Other host []{#Chapter_5.xhtml#idx_deb284d1 .index-entry
index-entry="SELinux:considerations for mounts"}directories accessible
by default are `/etc`{.inlineCode} as read-only and `/usr`{.inlineCode}
as read/execute.[]{.sentence-end} Also, resources under
`/var/lib/containers/overlay/`{.inlineCode} are labeled as
`container_share_t`{.inlineCode}.

What happens if we try to mount a directory that is not correctly
labeled?

Podman still executes the container without complaining about the wrong
labeling, but the mounted directory or file will not be accessible from
a process running inside the containers, which are labeled with the
`container_t`{.inlineCode} context type.[]{.sentence-end} The following
example tries to mount a custom document root for an
`n`{.inlineCode}`ginx`{.inlineCode} container without respecting the
labeling constraints:

``` {.programlisting .snippet-con}
$ mkdir ~/custom_docroot
$ echo "Hello World!" > ~/custom_docroot/index.html
$ podman run -d \
   --name custom_nginx \
  -p 8080:80 \
   -v ~/custom_docroot:/usr/share/nginx/html \
   docker.io/library/nginx
```

Apparently, everything went fine -- the container started properly, and
the processes inside it are running, but if we try to contact the NGINX
server, we see the error:

``` {.programlisting .snippet-con}
$ curl localhost:8080
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.21.3</center>
</body>
</html>
```

`403 Forbidden`{.inlineCode} shows that the
`n`{.inlineCode}`ginx`{.inlineCode} process cannot access the
`index.html`{.inlineCode} page.[]{.sentence-end} To fix this error, we
have two options: put SELinux in *permissive mode* or relabel the
mounted resources.[]{.sentence-end} By
putting[]{#Chapter_5.xhtml#idx_60c9edd8 .index-entry
index-entry="host storage:SELinux considerations, for mounts"} SELinux
in permissive mode, it continues to track down the violations without
blocking them.[]{.sentence-end} Anyway, this is not a good
practice[]{#Chapter_5.xhtml#idx_c5db702e .index-entry
index-entry="SELinux:considerations for mounts"} and should be used only
when we cannot correctly troubleshoot access issues and need to put
SELinux out of the equation.

The following command sets SELinux to permissive mode:

``` {.programlisting .snippet-con}
$ sudo setenforce 0
```

 note
**Important** **note**

Permissive mode is not equal to disabling SELinux
entirely.[]{.sentence-end} When working in this mode, SELinux still logs
**Access Vector Cache** (**AVC**)
denials[]{#Chapter_5.xhtml#idx_001dab30 .index-entry
index-entry="Access Vector Cache (AVC)"} without
blocking.[]{.sentence-end} System admins can immediately switch between
permissive and enforcing mode without rebooting.[]{.sentence-end}
Disabling, on the other hand, implies a full system reboot.


Alternatively, you can disable SELinux for a single container using
`--security-opt label=disable`{.inlineCode}.

The second, and preferred, option is to simply relabel the resources we
need to mount.[]{.sentence-end} To achieve this result, we could use
SELinux command-line tools.[]{.sentence-end} As a shortcut, Podman
offers a simpler way -- the `:z`{.inlineCode} and `:Z`{.inlineCode}
suffixes applied to the volume mount arguments.[]{.sentence-end} The
difference between the two suffixes is subtle:

-   The `:z`{.inlineCode} suffix tells Podman to relabel the mounted
    resources in order to enable all containers to read and write on the
    storage.[]{.sentence-end} It works with both volumes and bind
    mounts.
-   The `:Z`{.inlineCode} suffix tells Podman to relabel the mounted
    resources in order to enable only the current container to read and
    write on the storage exclusively.[]{.sentence-end} This also works
    with both volumes and bind mounts.

Be cautious when using these flags on system-critical directories (such
as `/etc`{.inlineCode} or `/home`{.inlineCode}), as relabeling these
files can prevent other host processes or the operating system from
accessing them correctly.

To test the[]{#Chapter_5.xhtml#idx_1463562a .index-entry
index-entry="host storage:SELinux considerations, for mounts"}
difference, let\'s try to run the container again with the
`:z`{.inlineCode} suffix and see what happens:

``` {.programlisting .snippet-con}
$ podman run -d \
   --name custom_nginx \
  –p 8080:80 \
   -v ~/custom_docroot:/usr/share/nginx/html:z \
   docker.io/library/nginx
```

Now, the []{#Chapter_5.xhtml#idx_880fb2de .index-entry
index-entry="SELinux:considerations for mounts"}HTTP calls return the
expected results since the process was able to access the
`index.html`{.inlineCode} file without being blocked by SELinux:

``` {.programlisting .snippet-con}
$ curl localhost:8080
Hello World!
```

Let\'s look at the SELinux file context automatically applied to the
mounted directory:

``` {.programlisting .snippet-con}
$ ls -alZ ~/custom_docroot
total 20
drwxrwxr-x.  2 packt packt system_u:object_r:container_file_t:s0  4096 Oct 16 15:53 .
drwxrwxr-x. 74 packt packt unconfined_u:object_r:user_home_dir_t:s0 12288 Oct 16 16:32 ..
-rw-rw-r--.  1 packt packt system_u:object_r:container_file_t:s0  13 Oct 16 15:53 index.html
```

Let\'s focus on the `system_u:object_r:container_file_t:s0`{.inlineCode}
label.[]{.sentence-end} The final `s0`{.inlineCode} field is a
**Multi-Level Security** (**MLS**) sensitivity level,
which[]{#Chapter_5.xhtml#idx_3c95e246 .index-entry
index-entry="Multi-Level Security (MLS) sensitivity level"} means that
all processes with the same sensitivity level will have read/write
access to the resource.[]{.sentence-end} Therefore, other containers
that run with the `s0`{.inlineCode} sensitivity level will be able to
mount the resource with read/write access privileges.[]{.sentence-end}
This also represents a security issue since a malicious container on the
same host would be able to attack other containers by stealing or
overwriting data.

The solution to this problem[]{#Chapter_5.xhtml#idx_2c72df84
.index-entry index-entry="Multi-Category Security (MCS)"} is called
**Multi-Category Security** (**MCS**).[]{.sentence-end} SELinux uses MCS
to configure additional categories, which are plaintext labels applied
to the resources along with the other []{#Chapter_5.xhtml#idx_ee5aae1c
.index-entry index-entry="SELinux:considerations for mounts"}SELinux
labels.[]{.sentence-end} MCS-labeled objects are then accessible only to
processes with the same categories assigned.

When a container is[]{#Chapter_5.xhtml#idx_2f65e214 .index-entry
index-entry="host storage:SELinux considerations, for mounts"} started,
processes inside it are labeled with MCS categories, following the
`cXXX,cYYY`{.inlineCode} pattern, where `XXX`{.inlineCode} and
`YYY`{.inlineCode} are randomly picked integers.

Podman automatically applies MCS categories to mounted resources when
`Z`{.inlineCode} (uppercase) is passed.[]{.sentence-end} To test this
behavior, let\'s again run the `n`{.inlineCode}`ginx`{.inlineCode}
container with the `:Z`{.inlineCode} suffix:

``` {.programlisting .snippet-con}
$ podman run -d \
   --name custom_nginx \
  –p 8080:80 \
   -v ~/custom_docroot:/usr/share/nginx/html:Z \
   docker.io/library/nginx
```

We can immediately see that the mounted folder has been relabeled with
MCS categories:

``` {.programlisting .snippet-con}
$ ls -alZ ~/custom_docroot
total 20
drwxrwxr-x.  2 packt packt system_u:object_r:container_file_t:s0:c16,c898  4096 Oct 16 15:53 .
drwxrwxr-x. 74 packt packt unconfined_u:object_r:user_home_dir_t:s0       12288 Oct 16 21:12 ..
-rw-rw-r--.  1 packt packt system_u:object_r:container_file_t:s0:c16,c898    13 Oct 16 15:53 index.html
```

A simple test will return the expected `Hello World!`{.inlineCode} text,
proving that the processes inside the container are allowed to access
the target resources:

``` {.programlisting .snippet-con}
$ curl localhost:8080
Hello World!
```

What happens if we run a second container with the same approach, by
applying `:Z`{.inlineCode} again to the same bind mount?

``` {.programlisting .snippet-con}
$ podman run -d \
   --name custom_nginx2 \
  –p 8081:80 \
   -v ~/custom_docroot:/usr/share/nginx/html:Z \
   docker.io/library/nginx
```

This time, we []{#Chapter_5.xhtml#idx_6efea669 .index-entry
index-entry="host storage:SELinux considerations, for mounts"}run the
HTTP test on port `8081`{.inlineCode}, and `HTTP GET`{.inlineCode} still
works correctly:

``` {.programlisting .snippet-con}
$ curl localhost:8081
Hello World!
```

However, if we []{#Chapter_5.xhtml#idx_ba8b1668 .index-entry
index-entry="SELinux:considerations for mounts"}test once again the
container mapped to port `8080`{.inlineCode}, we will get an unexpected
`403 Forbidden`{.inlineCode} message:

``` {.programlisting .snippet-con}
$ curl localhost:8080
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.21.3</center>
</body>
</html>
```

Not surprisingly, the second container was executed with the
`:Z`{.inlineCode} suffix and relabeled the directory with a new pair of
MCS categories, thus making the first container unable to access the
previously available content.

 packt_tip
**Important** **note**

The previous examples were conducted with bind mounts, but they still
apply to volumes in the same way.[]{.sentence-end} Use these techniques
with caution to avoid unwanted relabels of a bind-mounted system
or[]{#Chapter_5.xhtml#idx_96eee265 .index-entry
index-entry="SELinux:considerations for mounts"}
home[]{#Chapter_5.xhtml#idx_b7925509 .index-entry
index-entry="host storage:SELinux considerations, for mounts"}
directories.[]{.sentence-end} In addition to that, you should not use
`:z`{.inlineCode} and `:Z`{.inlineCode} with volumes; Podman
automatically handles their SELinux labeling.


In this subsection, we demonstrated the power of SELinux to manage
containers and resource isolation.[]{.sentence-end} Let\'s conclude this
chapter with an overview of other types of storage that can be attached
to containers.

# Attaching other types of storage to a container {#Chapter_5.xhtml#h1_154 .heading-1}

Along with bind mounts and volumes, it is possible to attach other types
of storage to containers, more specifically, of the
`tmpfs`{.inlineCode}, `image`{.inlineCode}, and `devpts`{.inlineCode}
kinds.

## Attaching tmpfs storage {#Chapter_5.xhtml#h2_155 .heading-2}

Sometimes, we[]{#Chapter_5.xhtml#idx_d406e0d1 .index-entry
index-entry="containers:tmpfs storage, attaching"} need to attach
storage to containers that are not meant to be persistent (for example,
cache usage).[]{.sentence-end} Using volumes or bind mounts would
clutter the host local disk (or any other backend if using different
storage drivers).[]{.sentence-end} In those
[]{#Chapter_5.xhtml#idx_25db6efb .index-entry
index-entry="tmpfs storage:attaching, to container"}particular cases, we
can use a `tmpfs`{.inlineCode} volume.

`tmpfs`{.inlineCode} is a virtual memory filesystem, which means that
all its contents are created inside the host virtual
memory.[]{.sentence-end} A benefit of `tmpfs`{.inlineCode} is that it
provides faster I/O since all the read/write operations mostly happen in
the RAM.

To attach a `tmpfs`{.inlineCode} volume to a container, we can use the
`--mount`{.inlineCode} option or the `--tmpfs`{.inlineCode} option.

The `--mount`{.inlineCode} flag has the great advantage of being more
verbose and expressive regarding the storage type, source, destination,
and extra mount options.[]{.sentence-end} The following example runs an
`httpd`{.inlineCode} container with a `tmpfs`{.inlineCode} volume
attached to the container:

``` {.programlisting .snippet-con}
$ podman run –d –p 8080:80 \
   --name tmpfs_example1 \
   --mount type=tmpfs,tmpfs-size=512M,destination=/tmp \
   docker.io/library/httpd
```

The preceding []{#Chapter_5.xhtml#idx_ee39dbf7 .index-entry
index-entry="containers:tmpfs storage, attaching"}command creates a
`tmpfs`{.inlineCode} volume of 512 MB and
mounts[]{#Chapter_5.xhtml#idx_35fc1223 .index-entry
index-entry="tmpfs storage:attaching, to container"} it on the
`/tmp`{.inlineCode} folder of the container.[]{.sentence-end} We can
test the correct mount creation by running the `mount`{.inlineCode}
command inside the container:

``` {.programlisting .snippet-con}
$ podman exec -it tmpfs_example1 mount | grep '\/tmp'
tmpfs on /tmp type tmpfs (rw,nosuid,nodev,relatime,context="system_u:object_r:container_file_t:s0:c375,c804",size=524288k,uid=1000,gid=1000,inode64
```

This demonstrates that the `tmpfs`{.inlineCode} filesystem has been
correctly mounted inside the container.[]{.sentence-end} Stopping the
container will automatically discard `tmpfs`{.inlineCode}:

``` {.programlisting .snippet-con}
$ podman stop tmpfs_example1
```

The following example mounts a `tmpfs`{.inlineCode} volume using the
`--tmpfs`{.inlineCode} option:

``` {.programlisting .snippet-con}
$ podman run –d –p 8080:80 \
   --name tmpfs_example2 \
   --tmpfs /tmp:rw,size= 524288k,mode=1777 \
   docker.io/library/httpd
```

This example provides the same results as the previous one: a running
container with a 512 MB `tmpfs`{.inlineCode} volume mounted on the
`/tmp`{.inlineCode} directory in read/write mode and 1,777 permissions.

By default, `tmpfs`{.inlineCode} is mounted inside the container with
the following mount options: `rw`{.inlineCode}, `noexec`{.inlineCode},
`nosuid`{.inlineCode}, and `nodev`{.inlineCode}.

Another interesting feature is the automatic MCS labeling from
SELinux.[]{.sentence-end} This provides automatic segregation of the
filesystem and prevents any other container from accessing the data in
memory.

## Attaching images {#Chapter_5.xhtml#h2_156 .heading-2}

OCI images[]{#Chapter_5.xhtml#idx_82445572 .index-entry
index-entry="containers:images, attaching"} are the base that provides
layers and metadata to start containers, but they can also be attached
to a container filesystem at runtime.[]{.sentence-end}
This[]{#Chapter_5.xhtml#idx_e9fb825b .index-entry
index-entry="OCI images:attaching, to container"} can be useful for
troubleshooting purposes, for attaching binaries that are available in a
foreign image, or to perform security scans on images.[]{.sentence-end}
When an OCI image is mounted inside a container, an extra overlay is
created.[]{.sentence-end} This implies that even when the image is
mounted with read/write permissions, users never alter the original
image but the upper overlay only.

The following example mounts a `busybox`{.inlineCode} image with
read/write permissions inside an Alpine container:

``` {.programlisting .snippet-con}
$ podman run -it \
   --mount type=image,src=docker.io/library/busybox,dst=/mnt,rw=true \
   Alpine
```

 packt_tip
**Important** **note**

Any image you intend to mount must be locally available on the host
first.[]{.sentence-end} Unlike the standard `podman run`{.inlineCode}
command, which automatically fetches a missing base image, Podman
requires mountable images to be pulled beforehand.[]{.sentence-end}
Running a preliminary `podman pull`{.inlineCode} command ensures that
the image is cached and ready for use.


## Attaching devpts {#Chapter_5.xhtml#h2_157 .heading-2}

This option[]{#Chapter_5.xhtml#idx_b07b627f .index-entry
index-entry="containers:devpts, attaching"} is useful for attaching a
**pseudo terminal slave** (**PTS**) to a
[]{#Chapter_5.xhtml#idx_b07bb38d .index-entry
index-entry="pseudo terminal slave (PTS)"}container.[]{.sentence-end}
This[]{#Chapter_5.xhtml#idx_4c772f6c .index-entry
index-entry="devpts:attaching, to container"} feature was introduced in
Podman 2.1.0 to support containers that need to mount
`/dev/`{.inlineCode} from the host into the container, while still
creating a terminal.[]{.sentence-end} The `/dev`{.inlineCode}
pseudo-filesystem of the host enables containers to gain direct access
to the machine\'s physical or virtual devices.

To create a container with the `/dev`{.inlineCode} filesystem and a
`devpts`{.inlineCode} device attached, run the following command:

``` {.programlisting .snippet-con}
$ sudo podman run -it \
  -v /dev/:/dev:rslave \
  --mount type=devpts,destination=/dev/pts \
  docker.io/library/fedora
```

To check the[]{#Chapter_5.xhtml#idx_a1feff28 .index-entry
index-entry="devpts:attaching, to container"} result of the
`mount`{.inlineCode} option, we require an extra tool inside the
container.[]{.sentence-end} For this reason, we can install it with the
following command:

``` {.programlisting .snippet-con}
[root@034c8a61a4fc /]# dnf install -y toolbox
```

The[]{#Chapter_5.xhtml#idx_7e75feb6 .index-entry
index-entry="containers:devpts, attaching"} resulting container has an
extra, non-isolated, `devpts`{.inlineCode} device mounted on
`/dev/pts`{.inlineCode}:

``` {.programlisting .snippet-con}
# mount | grep '\/dev\/pts'
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,seclabel,gid=5,mode=620,ptmxmode=000)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,context="system_u:object_r:container_file_t:s0:c299,c741",gid=5,mode=620,ptmxmode=666)
```

The preceding output was extracted by running the `mount`{.inlineCode}
command inside the container.

# Summary {#Chapter_5.xhtml#h1_158 .heading-1}

In this chapter, we have completed a journey on container storage and
Podman features offered to manipulate it.[]{.sentence-end} The material
in this chapter is crucial to understanding how Podman manages both
ephemeral and persistent data and provides best practices to users to
manipulate their data.

In the first section, we learned why container storage matters and how
it should be correctly managed both in single-host and orchestrated,
multi-host environments.

In the second section, we took a deep dive into container storage
features and storage drivers, with a special focus on overlayfs.

In the third section, we learned how to copy files to and from a
container.[]{.sentence-end} We also saw how changes could be committed
to a new image.

The fourth section described the different possible scenarios of storage
attached to a container, covering bind mounts, volumes,
`tmpfs`{.inlineCode}, images, and
`devpts`{.inlineCode}.[]{.sentence-end} This section was also a perfect
fit to discuss SELinux interaction in storage management and see how we
can use it to isolate storage resources across containers on the same
host.

In the next chapter, we will learn about a very important topic for both
developers and operations teams, which is how to build OCI images with
both Podman and **Buildah**, an advanced and specialized image-building
tool.

# Further reading {#Chapter_5.xhtml#h1_159 .heading-1}

Refer to the following resources for more information:

-   Containers/storage project page:
    [[https://github.com/containers/storage]{.url}](https://github.com/containers/storage){style="text-decoration: none;"}
-   *Container Labeling*:
    [[https://danwalsh.livejournal.com/81269.html]{.url}](https://danwalsh.livejournal.com/81269.html){style="text-decoration: none;"}
-   *Why* *you should be using* *Multi-Category Security* *for your
    Linux containers*:
    [[https://www.redhat.com/en/blog/why-you-should-be-using-multi-category-security-your-linux-containers]{.url}](https://www.redhat.com/en/blog/why-you-should-be-using-multi-category-security-your-linux-containers){style="text-decoration: none;"}
-   Udica: Generate SELinux policies:
    [[https://github.com/containers/udica]{.url}](https://github.com/containers/udica){style="text-decoration: none;"}
-   Overlay source code:
    [[https://github.com/containers/storage/blob/main/drivers/overlay/overlay.go]{.url}](https://github.com/containers/storage/blob/main/drivers/overlay/overlay.go){style="text-decoration: none;"}

# Get This Book's PDF Version and Exclusive Extras {#Chapter_5.xhtml#h1_160 .heading-1}

Scan the QR code (or go to
[[packtpub.com/unlock]{.url}](https://packtpub.com/unlock){style="text-decoration: none;"}).[]{.sentence-end}
Search for this book by name, confirm the edition, and then follow the
steps on the page.

![Image](images/B31467_5_1.png){style="width:25%;"}

![Image](images/B31467_5_2.png){style="width:25%;"}

*Note: Keep your invoice handy.[]{.sentence-end} Purchases made directly
from Packt don't require one.*


[]{#Part_2.xhtml}

 {.section .part}