# Part 3 {#Part_3.xhtml#h1_271 .partNumber}

# Managing and Integrating Containers Securely {#Part_3.xhtml#h1_272 .partTitle}

In this part of the book, you will learn how to troubleshoot, monitor,
and create advanced configurations for the running containers on the
operating system host.

This part of the book includes the following chapters:

-   *[Chapter 11](#Chapter_11.xhtml#h1_273){.chapref}*, *Troubleshooting
    and Monitoring Containers*
-   *[Chapter 12](#Chapter_12.xhtml#h1_295){.chapref}*, *Implementing
    Container Networking Concepts*
-   *[Chapter 13](#Chapter_13.xhtml#h1_314){.chapref}*, *Docker*
    *Migration Tips and Tricks*
-   *[Chapter 14](#Chapter_14.xhtml#h1_330){.chapref}*, *Interacting
    with* *systemd* *and* *Kubernetes*
-   *[Chapter 15](#Chapter_15.xhtml#h1_348){.chapref}*, *Managing*
    *Your* *Container,* *Kubernetes, and AI Workloads from a Graphical
    Interface*


[]{#Chapter_11.xhtml}

 {.section .chapter-first}
# 11 {#Chapter_11.xhtml#h1_273 .chapterNumber}

# Troubleshooting and Monitoring Containers {#Chapter_11.xhtml#h1_274 .chapterTitle}

Running a container could be mistaken as the ultimate goal for a DevOps
team, but instead, this is only the first step of a long
journey.[]{.sentence-end} System administrators should ensure that their
systems are working properly to keep services up and running; in the
same way, the DevOps team should ensure that their containers are
working properly.

In container management activities, having the right knowledge of
troubleshooting techniques could really help minimize any impact on the
final services, reducing downtime.[]{.sentence-end} Talking of issues
and troubleshooting, a good practice is to keep monitoring containers to
easily intercept any issues or errors to speed up recovery.

In this chapter, we\'re going to cover the following main topics:

-   Troubleshooting running containers
-   Monitoring containers with health checks
-   Inspecting our container build results
-   Advanced troubleshooting with `nsenter`{.inlineCode}

# Technical requirements {#Chapter_11.xhtml#h1_275 .heading-1}

Before proceeding with the chapter information and examples, a machine
with a working Podman installation is required.[]{.sentence-end} As
stated in *Chapter* *3*, *Running* *the* *First* *Container*, all the
examples in the book are executed on a Fedora 40 system or later, but
can be reproduced on your OS of choice.

A good understanding of the topics covered in *Chapter* *4*, *Managing*
*Running* *Containers*, and *Chapter* *5*, *Implementing* *Storage*
*for* *the* *Container\'s* *Data*, will be useful to easily grasp
concepts relating to container registries.

# Troubleshooting running containers {#Chapter_11.xhtml#h1_276 .heading-1}

Troubleshooting []{#Chapter_11.xhtml#idx_b463104f .index-entry
index-entry="containers:troubleshooting"}containers is an important
practice that we need experience with to solve common issues and
investigate any bugs we may encounter on the container layer or in the
application running inside our containers.

Starting from *Chapter* *3*, *Running* *the* *First* *Container*, we
started working with basic Podman commands for running and then
inspecting containers on our host system.[]{.sentence-end} We saw how we
can collect logs with the
`podman`{.inlineCode}` `{.inlineCode}`logs`{.inlineCode} command, and we
also learned how to use the information provided by the
`podman`{.inlineCode}` `{.inlineCode}`inspect`{.inlineCode}
command.[]{.sentence-end} Finally, we should also consider taking a look
at the output of the useful
`podman`{.inlineCode}` `{.inlineCode}`system`{.inlineCode}` `{.inlineCode}`df`{.inlineCode}
command, which will report storage usage for our containers and images,
and also the useful
`podman`{.inlineCode}` `{.inlineCode}`info`{.inlineCode} command, which
will show useful information on the host where we are running Podman.

In general, we should always consider that the running container is just
a process on the host system, so we always have all the tools and
commands available for troubleshooting the underlying OS and its
available resources.

A best practice for troubleshooting containers could be a top-down
approach, analyzing the application layer first, then moving to the
container layer, and finally down to the base host system.

At the container level, many of the issues that we may encounter have
been summarized by the Podman project team in a comprehensive list on
the project page (refer to the link shared in the *Further reading*
section).[]{.sentence-end} We will cover some of the more useful ones in
the following sections.

## Permission denied while using storage volumes {#Chapter_11.xhtml#h2_277 .heading-2}

A very common issue that we may[]{#Chapter_11.xhtml#idx_15d42a5d
.index-entry index-entry="storage permissions:issues"} encounter during
our activities on RHEL, Fedora, or any Linux distribution that uses the
SELinux security subsystem is related to storage
permissions.[]{.sentence-end} The error described as follows is
triggered when SELinux is set to `Enforcing`{.inlineCode} mode, which is
also the suggested approach to fully guarantee the mandatory access
security features of SELinux.

### The SELinux layer: security contexts {#Chapter_11.xhtml#h3_278 .heading-3}

The first barrier is often the[]{#Chapter_11.xhtml#idx_6566c16e
.index-entry index-entry="SELinux layer"} SELinux security
subsystem.[]{.sentence-end} SELinux uses labels (extended filesystem
attributes) to ensure that processes in one container cannot *break out*
and access files belonging to the host []{#Chapter_11.xhtml#idx_0588219b
.index-entry index-entry="security contexts"}or other containers.

By default, if you create a directory on your host and mount it into a
container, the containerized process will not have the correct SELinux
label to interact with it.

``` {.programlisting .snippet-con}
$ mkdir ~/mycontent
$ podman run -v ~/mycontent:/content fedora \
touch /content/file
touch: cannot touch '/content/file': Permission denied
```

As we can see, the `touch`{.inlineCode} command reports a
`Permission`{.inlineCode}` `{.inlineCode}`denied`{.inlineCode} error,
because actually, it cannot write in the filesystem.

As we saw in detail in *Chapter* *5*, *Implementing* *Storage* *for*
*the* *Container\'s* *Data*, SELinux recursively applies labels to files
and directories to define their context.[]{.sentence-end} Those labels
are usually stored as extended filesystem attributes.[]{.sentence-end}
SELinux uses contexts to manage policies and define which processes can
access a specific resource.

The container we just ran got its own Linux namespace and an SELinux
label that is completely different from the local user in the Fedora
workstation, which is why we actually got that error before.

Without a proper label, the SELinux system prevents the processes
running in the container from accessing the content.[]{.sentence-end}
This is also because Podman does not change the labels set by the OS if
not explicitly requested through a command option.

#### The fix: relabeling with :z and :Z {#Chapter_11.xhtml#h4_279 .heading-4}

To let Podman change the label for a[]{#Chapter_11.xhtml#idx_f58dd7bd
.index-entry index-entry="storage permissions:fix"} container, we can
use either of two suffixes, `:z`{.inlineCode} or `:Z`{.inlineCode}, for
the volume mount.[]{.sentence-end} These options tell Podman to relabel
file objects on the volume.

To tell Podman to automatically adjust the SELinux labels for the
mounted volume, we use the `:z`{.inlineCode} or `:Z`{.inlineCode}
suffixes:

-   `:z`{.inlineCode} (Shared): Tells Podman that the volume content
    will be shared among multiple containers.[]{.sentence-end} It
    applies a shared content label.

-   `:Z`{.inlineCode} (Private): Tells Podman to apply a private,
    unshared label.[]{.sentence-end} This is the more secure choice if
    only one container needs access to that specific
    data.[]{.sentence-end} The command would result in something like
    this:

    ``` {.programlisting .snippet-con-one}
    # Using :Z to grant exclusive access to the container
    $ podman run -v ~/mycontent:/content:Z fedora \
    touch /content/file
    ```

As we can see, the command didn\'t report any error; it worked.

### The rootless layer: user namespaces (UID/GID) {#Chapter_11.xhtml#h3_280 .heading-3}

Even if[]{#Chapter_11.xhtml#idx_83ed4f3d .index-entry
index-entry="storage permissions:issues"} you solve the SELinux issue,
you may still encounter
`P`{.inlineCode}`ermission denied`{.inlineCode}.[]{.sentence-end} In a
rootless environment, this is usually due
to[]{#Chapter_11.xhtml#idx_a5dbbb99 .index-entry
index-entry="user namespaces"} how Podman maps users between the host
and the container -- a feature called **user** **namespaces**.

In the world of []{#Chapter_11.xhtml#idx_c76b4068 .index-entry
index-entry="User IDs (UIDs)"}Linux containers, a user namespace is a
kernel feature that provides isolation of **User IDs** (**UIDs**) and
**Group IDs** (**GIDs**).[]{.sentence-end}
It[]{#Chapter_11.xhtml#idx_8b3066bd .index-entry
index-entry="Group IDs (GIDs)"} essentially allows a process to have a
different identity inside the namespace than it has on the host system.

In a rootless setup, you are *you* on the host (for example,
`UID`{.inlineCode}` 1000`{.inlineCode}), but inside the container, you
usually appear as root
(`UID`{.inlineCode}` 0`{.inlineCode}).[]{.sentence-end} However, your
\"root\" identity inside the container is actually an alias for your
user on the host.[]{.sentence-end} If the container tries to run a
process as a different user (such as a `MySQL`{.inlineCode} or
`nginx`{.inlineCode} user with `UID`{.inlineCode}` 999`{.inlineCode}),
that UID might not have permission to write to the host folder you just
mounted.

#### Identifying the mismatch {#Chapter_11.xhtml#h4_281 .heading-4}

You can see this []{#Chapter_11.xhtml#idx_c9efe129 .index-entry
index-entry="storage permissions:issues"}mapping in action using
`podman`{.inlineCode}` top`{.inlineCode}.[]{.sentence-end} If you run a
container as a specific user, Podman maps that ID to a high-range
*sub-UID* on your host, defined
in` `{.inlineCode}`/`{.inlineCode}`etc`{.inlineCode}`/`{.inlineCode}`subuid`{.inlineCode}.

``` {.programlisting .snippet-con}
$ podman run -d --name web -v ~/data:/data:Z nginx
$ podman top web user,huser
USER    HUSER
root    1000      # Inside is root, outside is your local user
nginx   100098    # Inside is nginx (999), outside is a sub-UID
```

In this example, the `nginx`{.inlineCode} process inside the container
is actually running as host
`UID`{.inlineCode}` `{.inlineCode}`100098`{.inlineCode}.[]{.sentence-end}
Since your host folder `~/data`{.inlineCode} is likely owned by
`UID`{.inlineCode}` `{.inlineCode}`1000`{.inlineCode}, the
`nginx`{.inlineCode} user is blocked.

#### The fix: the :U flag {#Chapter_11.xhtml#h4_282 .heading-4}

The most efficient way[]{#Chapter_11.xhtml#idx_1a5de287 .index-entry
index-entry="storage permissions:fix"} to solve this in modern Podman,
version 3.1 and later, is the `:U`{.inlineCode} mount
option.[]{.sentence-end} This tells Podman to automatically
`chown`{.inlineCode}, change ownership, of the host directory to match
the UID and GID used inside the container.

``` {.programlisting .snippet-con}
$ podman run -v ~/mycontent:/content:Z,U fedora touch /content/file
```

#### The fix: podman unshare {#Chapter_11.xhtml#h4_283 .heading-4}

If you need to manually fix permissions for a host directory so a
rootless container can use it, you cannot simply use the standard
`sudo`{.inlineCode}` `{.inlineCode}`chown`{.inlineCode}.[]{.sentence-end}
You must use
`podman`{.inlineCode}` `{.inlineCode}`unshare`{.inlineCode}, which
executes a command inside the same user namespace the container uses.

``` {.programlisting .snippet-con}
# This sets the host directory to be owned by the internal 'root' of the container
$ podman unshare chown -R 0:0 ~/mycontent
```

#### The fix: \--userns=keep-id {#Chapter_11.xhtml#h4_284 .heading-4}

If your containerized application needs to see the exact same UID as
your host user, common for development tools, use the
`--`{.inlineCode}`userns`{.inlineCode}`=keep-id`{.inlineCode}
flag.[]{.sentence-end} This ensures that
`UID`{.inlineCode}` 1000`{.inlineCode} on the host remains
`UID`{.inlineCode}` 1000`{.inlineCode} inside the container.

``` {.programlisting .snippet-con}
$ podman run --userns=keep-id -v ~/mycontent:/content:Z fedora ...
```

Depending on the issue []{#Chapter_11.xhtml#idx_ef807ec4 .index-entry
index-entry="storage permissions:fix"}you encounter, you may have one or
multiple ways to overcome and fix the issue to continue to
work.[]{.sentence-end} Let\'s continue to see how to troubleshoot other
issues in the next section.

## Issues with the ping command in rootless containers {#Chapter_11.xhtml#h2_285 .heading-2}

On some hardened Linux []{#Chapter_11.xhtml#idx_5dda1f95 .index-entry
index-entry="ping command:issues"}systems, the `ping`{.inlineCode}
command execution could be limited to only a restricted group of
users.[]{.sentence-end} This could cause the failure of the
`ping`{.inlineCode} command used in a container.

As we saw in *Chapter* *3*, *Running* *the* *First* *Container*, when
starting the container, the base OS will associate with it a different
user ID from the one used in the container itself.[]{.sentence-end} The
user ID associated with the container might fall outside the allowed
range of user IDs permitted to use the `ping`{.inlineCode} command.

In a Fedora workstation installation, the default configuration will
allow any container to run the `ping`{.inlineCode} command without
issues.[]{.sentence-end} To manage restrictions on the usage of the
`ping`{.inlineCode} command, Fedora uses the
`ping_group_range`{.inlineCode} kernel parameter, which defines the
allowed system groups that can execute the `ping`{.inlineCode} command.

If we take a look at a just-installed Fedora workstation, the default
range is the following:

``` {.programlisting .snippet-con}
$ cat /proc/sys/net/ipv4/ping_group_range
0      2147483647
```

So, nothing to worry about for a brand-new Fedora
system.[]{.sentence-end} But what about if the range is smaller than
this one?

Well, we test this behavior by changing the allowed range with a simple
command.[]{.sentence-end} In this example, we are going to restrict the
range and see that the `ping`{.inlineCode} command will actually fail
then:

``` {.programlisting .snippet-con}
$ sudo sysctl -w "net.ipv4.ping_group_range=0 0"
```

Just in case the range is smaller than the one reported in the previous
output, we can make it persistent by adding a file to
`/`{.inlineCode}`etc`{.inlineCode}`/`{.inlineCode}`sysctl.d`{.inlineCode}
that contains
`net.ipv4.ping_group_range`{.inlineCode}`=0`{.inlineCode}` `{.inlineCode}`0`{.inlineCode}.

The applied change in the `ping`{.inlineCode} group range will impact
the mapped user privileges to run the `ping`{.inlineCode} command inside
the container.

Let\'s start by building a Fedora-based image with the
`iputils`{.inlineCode} package (not included by default) using Buildah:

``` {.programlisting .snippet-con}
$ container=$(buildah from docker.io/library/fedora) && \
  buildah run $container -- dnf install -y iputils && \
  buildah commit $container ping_example
```

We can test it by running the following command inside a container:

``` {.programlisting .snippet-con}
$ podman run --rm ping_example ping -W10 -c1 redhat.com
PING redhat.com (209.132.183.105): 56 data bytes
--- redhat.com ping statistics ---
1 packets transmitted, 0 packets received, 100% packet loss
```

The command, executed on a[]{#Chapter_11.xhtml#idx_9d932f14 .index-entry
index-entry="ping command:issues"} system with a restricted range,
produces a 100% packet loss since the `ping`{.inlineCode} command is not
able to send packets over a raw socket.

 note
**Important** **note**

Do not forget to restore the original `ping_group_range`{.inlineCode}
before proceeding with the next examples.[]{.sentence-end} On Fedora,
the default configuration can be restored with the
`sudo`{.inlineCode}` `{.inlineCode}`sysctl`{.inlineCode}` `{.inlineCode}`-w`{.inlineCode}` `{.inlineCode}`"`{.inlineCode}`net.ipv4.ping_group_range`{.inlineCode}`=0`{.inlineCode}` `{.inlineCode}`2147483647"`{.inlineCode}
command and by removing any persistent configuration applied under
`/`{.inlineCode}`etc`{.inlineCode}`/`{.inlineCode}`sysctl.d`{.inlineCode}
during the exercise.[]{.sentence-end} For a base container image that we
are building through a Dockerfile, we may need to add a brand-new user
with a large UID/GID.[]{.sentence-end} This will create a large, sparse
`/`{.inlineCode}`var`{.inlineCode}`/log/`{.inlineCode}`lastlog`{.inlineCode}
file, and this can cause the build to hang forever.[]{.sentence-end}
This issue is related to the Go language, which does not correctly
support sparse files, leading to the creation of this huge file in the
container image.


The example demonstrates how a restriction in
`ping_group_range`{.inlineCode} impacts the execution of
`ping`{.inlineCode} inside a rootless container.[]{.sentence-end} By
setting the range to a value large enough to include the user\'s private
group GID (or one of the user\'s secondary groups), the
`ping`{.inlineCode} command will be able to send ICMP
packets[]{#Chapter_11.xhtml#idx_25c835d4 .index-entry
index-entry="ping command:issues"} correctly.

 note
**Good** **to** **know**

The
`/`{.inlineCode}`var`{.inlineCode}`/log/`{.inlineCode}`lastlog`{.inlineCode}
file is a binary and sparse file that contains information about the
last time that users logged in to the system.[]{.sentence-end} The
apparent size of a sparse file reported by
`ls`{.inlineCode}` `{.inlineCode}`-l`{.inlineCode} is larger than the
actual disk usage.[]{.sentence-end} A sparse file attempts to use
filesystem space in a more efficient way, writing the metadata that
represents the empty blocks to disk instead of the empty space that
should be stored in the block.[]{.sentence-end} This will use less disk
space.


As mentioned in the early paragraphs of this section, the Podman team
has created a long but non-comprehensive list of common
issues.[]{.sentence-end} We strongly suggest taking a look at it if any
issues are encountered:
[[https://github.com/containers/podman/blob/main/troubleshooting.md]{.url}](https://github.com/containers/podman/blob/main/troubleshooting.md){style="text-decoration: none;"}.

Troubleshooting could be tricky, but the first step is always the
identification of an issue.[]{.sentence-end} For this reason, a
monitoring tool could help in alerting as soon as possible in the case
of issues.[]{.sentence-end} Let\'s see how to monitor containers with
health checks in the next section.

# Monitoring containers with health checks {#Chapter_11.xhtml#h1_286 .heading-1}

Podman supports the option []{#Chapter_11.xhtml#idx_9bec6470
.index-entry index-entry="containers:monitoring, with health checks"}to
add a health check to containers.[]{.sentence-end} We
will[]{#Chapter_11.xhtml#idx_0626c334 .index-entry
index-entry="health checks:containers, monitoring"} go into these health
checks in depth in this section, and how to use them.

A health check is a Podman feature that can help determine the health or
readiness of the process running in a container.[]{.sentence-end} It
could be as simple as checking that the container\'s process is running,
but also more sophisticated, such as verifying that both the container
and its applications are responsive using, for example, network
connections.

In the container world, there is a distinct difference between a
container that is running and a container that is
healthy.[]{.sentence-end} By default, Podman considers a container *up*
as long as its primary process (`PID`{.inlineCode}` 1`{.inlineCode})
hasn\'t exited.[]{.sentence-end} However, a web server process might
still be running even if it\'s deadlocked, out of memory, or unable to
connect to its database.

A health check is a proactive monitoring mechanism used to ensure that
the service inside the container is actually functioning as
intended.[]{.sentence-end} Instead of just checking if the process
exists, Podman executes a specific command inside the container at
regular intervals to verify its internal state.

While you can manually define a health check when starting a container,
it is important to note that many enterprise-grade images come with
health checks built directly into the Image configuration:

-   **Built-in** **health** **checks:** Many official images (like those
    for databases or specialized web services)
    include[]{#Chapter_11.xhtml#idx_2e7db330 .index-entry
    index-entry="health checks:built-in health checks"} a
    `HEALTHCHECK`{.inlineCode} instruction in their
    Containerfile.[]{.sentence-end} When you run these images, Podman
    automatically starts the health monitoring logic without any extra
    configuration from the administrator.
-   **Manual** **health** **checks**: You can override an image\'s
    default or add a new check using the
    `--health-`{.inlineCode}`cmd`{.inlineCode} flag.[]{.sentence-end}
    This is useful for custom applications where you might want to
    []{#Chapter_11.xhtml#idx_ec92b2e4 .index-entry
    index-entry="health checks:manual health checks"}check a specific
    API endpoint or the existence of a *lock* file.

When a health check is active, the container\'s status in
`podman`{.inlineCode}` `{.inlineCode}`ps`{.inlineCode} will move through
several phases:

-   **Starting**: The[]{#Chapter_11.xhtml#idx_e52d2165 .index-entry
    index-entry="health checks:starting"} container has just launched,
    and the initial delay, or grace period, is active
-   **Healthy**: The[]{#Chapter_11.xhtml#idx_3af01252 .index-entry
    index-entry="health checks:healthy"} health check command exited
    successfully -- exit code `0`{.inlineCode}
-   **Unhealthy**: The []{#Chapter_11.xhtml#idx_21756d99 .index-entry
    index-entry="health checks:unhealthy"}command failed a consecutive
    number of times, defined by the *retries* limit

A health check is []{#Chapter_11.xhtml#idx_312ba87f .index-entry
index-entry="containers:monitoring, with health checks"}made up of five
core components.[]{.sentence-end} The first
is[]{#Chapter_11.xhtml#idx_79baaeb2 .index-entry
index-entry="health checks:containers, monitoring"} the main element
that will instruct Podman on the particular check to execute; the others
are used for configuring the schedule of the health
check.[]{.sentence-end} Let\'s see these elements in detail:

-   **Command**: This is the command[]{#Chapter_11.xhtml#idx_7c6b19d6
    .index-entry index-entry="health checks:command"} that Podman will
    execute inside the target container.[]{.sentence-end} The health of
    the container and its process will be determined by waiting for
    either a success (return code `0`{.inlineCode}) or a failure (any
    other exit codes).[]{.sentence-end}

    If our container provides a web server, for example, our health
    check command could be something really simple, such as a
    `curl`{.inlineCode} command that will try to connect to the web
    server port to make sure it is responsive.

-   **Retries**: This defines the []{#Chapter_11.xhtml#idx_5a6920be
    .index-entry index-entry="health checks:retries"}number of
    consecutive failed commands that Podman has to execute before the
    container will be marked as unhealthy.[]{.sentence-end} If a command
    executes successfully, Podman will reset the retry counter.

-   **Interval**: This[]{#Chapter_11.xhtml#idx_4bf6ef66 .index-entry
    index-entry="health checks:interval"} option defines the interval
    time between health checks.[]{.sentence-end}

    Finding the right interval time could be really difficult and
    requires some trial and error.[]{.sentence-end} If we set it to a
    small value, then our system may spend a lot of time running the
    health checks.[]{.sentence-end} But if we set it to a large value,
    we may struggle and catch timeouts.[]{.sentence-end} This value can
    be defined with a widely used time format: `30s`{.inlineCode} or
    `1h5m`{.inlineCode}.

-   **Start** **period**: In this[]{#Chapter_11.xhtml#idx_494b45e3
    .index-entry index-entry="health checks:start period"} period,
    Podman will ignore health check failures.[]{.sentence-end}

    We can consider this as a grace period that should be used to allow
    our application to be up and start replying correctly to any
    clients, as well as to our health checks.

-   **Timeout**: This defines []{#Chapter_11.xhtml#idx_cc156d6a
    .index-entry index-entry="health checks:timeout"}the period of time
    the health check itself must complete before being considered
    unsuccessful.[]{.sentence-end}

    ::: note-one
    Please note that `t`{.inlineCode}`imeout`{.inlineCode},
    `s`{.inlineCode}`tart period`{.inlineCode}`,`{.inlineCode} and
    `r`{.inlineCode}`etries`{.inlineCode} are optional, while command
    and interval must be set for a successful health check.
    :::

Let\'s take a look at a real example, supposing we want to define a
health check for a container and run that health check manually:

``` {.programlisting .snippet-con}
$ podman run -dt --name healthtest1 --health-cmd 'curl http://localhost || exit 1' --health-interval=0 quay.io/libpod/alpine_nginx:latest
89e3df8713d24fb56bfe3cfd545826d605581ead6ec1bec31c3b1363428355a2

$ podman ps
CONTAINER ID  IMAGE                               COMMAND               CREATED        STATUS                   PORTS       NAMES
89e3df8713d2  quay.io/libpod/alpine_nginx:latest  nginx -g daemon o...  7 seconds ago  Up 8 seconds (starting)  80/tcp      healthtest1
```

As we can see[]{#Chapter_11.xhtml#idx_d5fdb451 .index-entry
index-entry="containers:monitoring, with health checks"} from the
previous command block, we just started
[]{#Chapter_11.xhtml#idx_844ee3ea .index-entry
index-entry="health checks:containers, monitoring"}a brand-new container
named `health`{.inlineCode}`test1`{.inlineCode}, defining a
`healthcheck`{.inlineCode} command that will run the `curl`{.inlineCode}
command on the `localhost`{.inlineCode} address inside the target
container.[]{.sentence-end} Once the container started, it stayed on
state `starting`{.inlineCode} until we manually ran
`healthcheck`{.inlineCode}; let\'s try it with the following command.

``` {.programlisting .snippet-con}
$ podman healthcheck run healthtest1

$ echo $?
0

$ podman ps
CONTAINER ID  IMAGE                               COMMAND               CREATED         STATUS                   PORTS       NAMES
89e3df8713d2  quay.io/libpod/alpine_nginx:latest  nginx -g daemon o...  41 seconds ago  Up 41 seconds (healthy)  80/tcp      healthtest1
```

We can see from[]{#Chapter_11.xhtml#idx_a3c55909 .index-entry
index-entry="containers:monitoring, with health checks"} the previous
output that after manually[]{#Chapter_11.xhtml#idx_890bfe93 .index-entry
index-entry="health checks:containers, monitoring"} running
`healthcheck`{.inlineCode}, its exit code was `0`{.inlineCode}, meaning
that the check completed successfully and our container is marked as
`healthy`{.inlineCode}.[]{.sentence-end} In the previous example, we
also used the
`--`{.inlineCode}`healthcheck`{.inlineCode}`-interval=0`{.inlineCode}
option to actually disable the run interval and make the health check
manual.

 note
Please note that the previous example is only required when
`--health-interval=0`{.inlineCode} is specified, preventing Podman from
automatically scheduling the health check.[]{.sentence-end} In
real-world usage, Podman will run the health check automatically, and
the
`podman`{.inlineCode}` `{.inlineCode}`healthcheck`{.inlineCode}` run`{.inlineCode}
command will never need to be run by the user.


Podman uses **systemd** timers[]{#Chapter_11.xhtml#idx_bc8713a6
.index-entry index-entry="systemd timers"} to schedule health
checks.[]{.sentence-end} For this reason, it is mandatory if we want to
schedule a health check for our containers.[]{.sentence-end} Of course,
if some of our systems do not use systemd as the default daemon manager,
we could use different tools, such as `cron`{.inlineCode}, to schedule
the health checks, but these should be set manually.

Let\'s inspect how this automatic integration with systemd works by
creating a health check with an interval:

``` {.programlisting .snippet-con}
$ podman run -dt --name healthtest2 --health-cmd 'curl http://localhost || exit 1' --healthcheck-interval=10s quay.io/libpod/alpine_nginx:latest
69f3ca0ce7aa96401363db07daefe29184a1ede49f242cc4d3f8ab13a50ac5f6
$ podman ps
CONTAINER ID  IMAGE                               COMMAND               CREATED        STATUS                  PORTS       NAMES
69f3ca0ce7aa  quay.io/libpod/alpine_nginx:latest  nginx -g daemon o...  4 seconds ago  Up 4 seconds (healthy)  80/tcp      healthtest2  
```

As we can see from the previous code block, we just started a brand-new
container named `healthtest2`{.inlineCode}, defining the same
`health-`{.inlineCode}`cmd`{.inlineCode} of the previous example but now
specifying the `--health-interval=`{.inlineCode}`10s`{.inlineCode}
option to actually schedule the check every 10 seconds.

After the `podman`{.inlineCode}` `{.inlineCode}`run`{.inlineCode}
command, we also ran the
`podman`{.inlineCode}` `{.inlineCode}`ps`{.inlineCode} command to
actually inspect whether the health check is working properly, and as we
can see in the output, we have the `healthy`{.inlineCode} status for our
[]{#Chapter_11.xhtml#idx_5cc04d55 .index-entry
index-entry="containers:monitoring, with health checks"}brand-new
container.

But how does this []{#Chapter_11.xhtml#idx_ccb4d201 .index-entry
index-entry="health checks:containers, monitoring"}integration work?
Let\'s grab the container ID and search for it in the following
directory:

``` {.programlisting .snippet-con}
$ ls /run/user/$UID/systemd/transient/69f3*
/run/user/1000/systemd/transient/69f3ca0ce7aa96401363db07daefe29184a1ede49f242cc4d3f8ab13a50ac5f6-6433a92c3f3c8479.service
/run/user/1000/systemd/transient/69f3ca0ce7aa96401363db07daefe29184a1ede49f242cc4d3f8ab13a50ac5f6-6433a92c3f3c8479.timer
```

The directory shown in the previous example holds all the systemd
resources in use for our current user.[]{.sentence-end} In particular,
we looked into the `transient`{.inlineCode} directory, which holds
temporary unit files for our current user.

When we start a container with a health check and a schedule interval,
Podman will perform a transient setup of a systemd service and timer
unit file.[]{.sentence-end} This means that these unit files are not
permanent and can be lost on reboot, but recreated next time the
container restarts.

Let\'s inspect what is defined inside these files:

``` {.programlisting .snippet-con}
$ cat /run/user/1000/systemd/transient/69f3ca0ce7aa96401363db07daefe29184a1ede49f242cc4d3f8ab13a50ac5f6-6433a92c3f3c8479.service
# This is a transient unit file, created programmatically via the systemd API. Do not edit.
[Unit]
Description=/usr/bin/podman healthcheck run 69f3ca0ce7aa96401363db07daefe29184a1ede49f242cc4d3f8ab13a50ac5f6

[Service]
LogLevelMax=5
Environment="PATH=/home/vagrant/.local/bin:/home/vagrant/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin"
ExecStart=
ExecStart="/usr/bin/podman" "healthcheck" "run" "69f3ca0ce7aa96401363db07daefe29184a1ede49f242cc4d3f8ab13a50ac5f6"                                            

$ cat /run/user/1000/systemd/transient/69f3ca0ce7aa96401363db07daefe29184a1ede49f242cc4d3f8ab13a50ac5f6-6433a92c3f3c8479.timer
# This is a transient unit file, created programmatically via the systemd API. Do not edit.
[Unit]
Description=/usr/bin/podman healthcheck run 69f3ca0ce7aa96401363db07daefe29184a1ede49f242cc4d3f8ab13a50ac5f6

[Timer]
OnUnitInactiveSec=10s
AccuracySec=1s
```

As we can see []{#Chapter_11.xhtml#idx_26741156 .index-entry
index-entry="containers:monitoring, with health checks"}from the
previous example, the service unit file
[]{#Chapter_11.xhtml#idx_7d08ffcf .index-entry
index-entry="health checks:containers, monitoring"}contains the Podman
`healthcheck`{.inlineCode} command, while the timer unit file defines
the scheduling interval.

Finally, just because we may want a quick way to identify healthy or
unhealthy containers, we can use the following command to quickly output
them:

``` {.programlisting .snippet-con}
$ podman ps -a --filter health=healthy
CONTAINER ID  IMAGE                               COMMAND               CREATED         STATUS                   PORTS       NAMES
6a98d7bc448d  quay.io/libpod/alpine_nginx:latest  nginx -g daemon o...  18 seconds ago  Up 18 seconds (healthy)  80/tcp      healthtest1
05678f1bf846  quay.io/libpod/alpine_nginx:latest  nginx -g daemon o...  5 seconds ago   Up 4 seconds (healthy)   80/tcp      healthtest2
```

In this example, we used the
`--filter`{.inlineCode}` `{.inlineCode}`health=healthy`{.inlineCode}
option to display only the healthy containers with the
`podman`{.inlineCode}` `{.inlineCode}`ps`{.inlineCode} command.

By default, when a container\'s health check fails repeatedly, Podman
simply marks the container as unhealthy.[]{.sentence-end} While this is
useful for external monitoring tools, it doesn\'t solve the problem of a
stalled service.[]{.sentence-end} To bridge this gap, Podman provides
the `--health-on-failure`{.inlineCode} flag, allowing the container
engine to take immediate, automated action when a health check fails.

This flag transforms []{#Chapter_11.xhtml#idx_5b19a711 .index-entry
index-entry="containers:monitoring, with health checks"}a passive health
check into an active self-healing []{#Chapter_11.xhtml#idx_ae24b0b8
.index-entry
index-entry="health checks:containers, monitoring"}mechanism.[]{.sentence-end}
You can choose from several different strategies, depending on how you
want the recovery to be handled:

-   **none** (default): Podman only updates the health status to
    unhealthy and takes no further action.
-   **restart**: Podman will automatically restart the
    container.[]{.sentence-end} This is the most common *self-healing*
    approach for services that may occasionally deadlock or leak
    resources.
-   **stop**: The container is simply stopped.[]{.sentence-end} This is
    useful in scenarios where a failing service might corrupt data if
    allowed to keep running.
-   **kill**: Podman sends a `SIGKILL`{.inlineCode} to immediately
    terminate the container process.

To run a web server that automatically restarts itself if it becomes
unresponsive, you would combine your health check command with the
restart policy:

``` {.programlisting .snippet-con}
$ podman run -d \
  --name self_healing_web \
  --health-cmd "curl -f http://localhost/ || exit 1" \
  --health-on-failure=restart \
  nginx
```

Using this approach ensures that your service maintains maximum uptime
without requiring manual intervention from a system
administrator.[]{.sentence-end} It effectively brings a
**mini-orchestrator** capability directly to the Podman CLI.

While defining health checks at runtime via the CLI is flexible, the
most robust way to ensure a service is monitored correctly is to embed
the health check within the container image itself.[]{.sentence-end} By
adding a `HEALTHCHECK`{.inlineCode} instruction to your Containerfile
(or Dockerfile), you encode the *definition of health* directly into
[]{#Chapter_11.xhtml#idx_094f3bfb .index-entry
index-entry="containers:monitoring, with health checks"}the
application\'s []{#Chapter_11.xhtml#idx_0a8c99d7 .index-entry
index-entry="health checks:containers, monitoring"}metadata.

This approach ensures that anyone pulling your image, whether they are a
developer on Fedora or an automated system in a production cluster,
benefits from the same monitoring logic without having to look up the
correct CLI flags.

Let\'s look at a Containerfile that builds a simple web service using
the Red Hat **Universal Base Image** (**UBI**) and
[]{#Chapter_11.xhtml#idx_90ad1e16 .index-entry
index-entry="Universal Base Image (UBI)"}includes a built-in health
check.

``` {.programlisting .snippet-code}
FROM registry.access.redhat.com/ubi9/ubi-minimal

# Install a simple web server (microdnf is the package manager for ubi-minimal)
RUN microdnf install -y python3 && microdnf clean all

# Define the health check: try to fetch the local index every 10 seconds
HEALTHCHECK --interval=10s --timeout=3s --start-period=5s \
  --retries=3 CMD curl -f http://localhost:8080/ || exit 1

EXPOSE 8080
CMD ["python3", "-m", "http.server", "8080"]
```

We learned how to troubleshoot and monitor our containers so far, but
what about the container build process? Let\'s discover more about
container build inspection in the next section.

# Inspecting your container build results {#Chapter_11.xhtml#h1_287 .heading-1}

In *Chapter* *6*, we discussed in []{#Chapter_11.xhtml#idx_8b3cc756
.index-entry index-entry="container build results:inspecting"}detail the
container build process and learned how to create custom images using
Dockerfiles/Containerfiles or Buildah-native commands.[]{.sentence-end}
We also illustrated how the second approach helps achieve a greater
degree of control over the build workflow.

This section helps provide some best practices to inspect the build
results and understand potentially related issues.

## Troubleshooting builds from Dockerfiles {#Chapter_11.xhtml#h2_288 .heading-2}

When using Podman or Buildah to run a []{#Chapter_11.xhtml#idx_de246c02
.index-entry
index-entry="builds:troubleshooting, from Dockerfiles"}build based on a
Dockerfile/Containerfile, the build process prints all the
instructions\' outputs and related errors to
[]{#Chapter_11.xhtml#idx_09e83c85 .index-entry
index-entry="Dockerfiles:builds, triubleshooting from"}the terminal
`stdout`{.inlineCode}.[]{.sentence-end} For all `RUN`{.inlineCode}
instructions, errors generated from the executed commands are propagated
and printed for debugging purposes.

Let\'s now try to test some potential build issues.[]{.sentence-end}
This is not an exhaustive list of errors; the purpose is to provide a
method to analyze the root cause.

The first example shows a minimal build where a `RUN`{.inlineCode}
instruction fails due to an error in the executed
command.[]{.sentence-end} Errors in `RUN`{.inlineCode} instructions can
cover a wide range of cases, but the general rule of thumb is the
following: the executed command returns an exit code, and if this is
non-zero, the build fails and the error, along with the exit status, is
printed.

In the next example, we use the `yum`{.inlineCode} command to install
the `httpd`{.inlineCode} package, but we have intentionally made a typo
in the package name to generate an error.[]{.sentence-end} Here is the
Dockerfile transcript:

`Chapter11`{.inlineCode}`/`{.inlineCode}`RUN_command_error`{.inlineCode}`/`{.inlineCode}`Dockerfile`{.inlineCode}

``` {.programlisting .snippet-code}
FROM registry.access.redhat.com/ubi8

# Update image and install httpd
RUN yum install -y htpd && yum clean all -y

# Expose the default httpd port 80
EXPOSE 80

# Run the httpd
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
```

If we try to execute[]{#Chapter_11.xhtml#idx_b4dd53e4 .index-entry
index-entry="builds:troubleshooting, from Dockerfiles"} the command, we
will get an error generated by[]{#Chapter_11.xhtml#idx_65b6b945
.index-entry index-entry="Dockerfiles:builds, triubleshooting from"} the
`yum`{.inlineCode} command not being able to find the missing
`htpd`{.inlineCode} package:

``` {.programlisting .snippet-con}
$ buildah build -t custom_httpd .
STEP 1/4: FROM registry.access.redhat.com/ubi8
STEP 2/4: RUN yum install -y htpd && yum clean all -y
Updating Subscription Management repositories.
Unable to read consumer identity

This system is not registered with an entitlement server. You can use subscription-manager to register.

Red Hat Universal Base Image 8 (RPMs) - BaseOS  3.9 MB/s | 796kB     00:00  
Red Hat Universal Base Image 8 (RPMs) - AppStre 6.2 MB/s | 2.6 MB     00:00  
Red Hat Universal Base Image 8 (RPMs) - CodeRea 171 kB/s |  16 kB     00:00  

No match for argument: htpd
Error: Unable to find a match: htpd
error building at STEP "RUN yum install -y htpd && yum clean all -y": error while running runtime: exit status 1
```

The first two lines print the error message generated by the
`yum`{.inlineCode} command, as in a standard command-line environment.

Next, Buildah (and, in the same way, Podman) produces a message to
inform us about the step that generated the error.[]{.sentence-end} This
message is managed in the `imagebuildah`{.inlineCode} package by the
stage executor, which handles, as the name indicates, the execution of
the build stages and their statuses.[]{.sentence-end} The source code
can be inspected in the Buildah repository on GitHub:
[[https://github.com/containers/buildah/blob/main/imagebuildah/stage_executor.go]{.url}](https://github.com/containers/buildah/blob/main/imagebuildah/stage_executor.go){style="text-decoration: none;"}.

The message includes the Dockerfile instruction and the generated error,
along with the exit status.

The last line includes the error and the final exit status,
`1`{.inlineCode}, related to the `buildah`{.inlineCode} command
execution.

**Solution**: Use the error message to find the `RUN`{.inlineCode}
instruction that contains the failing command and fix or troubleshoot
the command error.

Another very common failure reason in builds is the missing parent
image.[]{.sentence-end} It could be related to a misspelled repository
name, a missing tag, or an unreachable registry.

The next example shows another variation of the previous Dockerfile,
where the image repository name is []{#Chapter_11.xhtml#idx_b00f5958
.index-entry
index-entry="builds:troubleshooting, from Dockerfiles"}mistyped and thus
does not exist in the remote []{#Chapter_11.xhtml#idx_3e49d9b3
.index-entry
index-entry="Dockerfiles:builds, triubleshooting from"}registry:

`Chapter11`{.inlineCode}`/`{.inlineCode}`FROM_repo_not_found`{.inlineCode}`/`{.inlineCode}`Dockerfile`{.inlineCode}

``` {.programlisting .snippet-code}
FROM registry.access.redhat.com/ubi_8

# Update image and install httpd
RUN yum install -y httpd && yum clean all -y

# Expose the default httpd port 80
EXPOSE 80

# Run the httpd
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
```

When running a build from this Dockerfile, we will encounter an error
caused by the missing image repository, as in the next example:

``` {.programlisting .snippet-con}
$ buildah build -t custom_httpd .
STEP 1/4: FROM registry.access.redhat.com/ubi_8
Trying to pull registry.access.redhat.com/ubi_8:latest...
error creating build container: initializing source docker://registry.access.redhat.com/ubi_8:latest: reading manifest latest in registry.access.redhat.com/ubi_8: name unknown: Repo not found
```

The last line []{#Chapter_11.xhtml#idx_44663f83 .index-entry
index-entry="builds:troubleshooting, from Dockerfiles"}produces a
different error.[]{.sentence-end} It is a very easy error
[]{#Chapter_11.xhtml#idx_81a5e943 .index-entry
index-entry="Dockerfiles:builds, triubleshooting from"}to troubleshoot
and only requires passing a valid repository to the `FROM`{.inlineCode}
instruction.

**Solution**: Fix the repository name and relaunch the build
process.[]{.sentence-end} Alternatively, verify that the target registry
holds the wanted repository.

What happens if we misspell the image tag? The next Dockerfile snippet
shows an invalid tag for the official Fedora image:

`Chapter11`{.inlineCode}`/`{.inlineCode}`FROM_tag_not_found`{.inlineCode}`/`{.inlineCode}`Dockerfile`{.inlineCode}

``` {.programlisting .snippet-code}
FROM docker.io/library/fedora:sometag

# Update image and install httpd
RUN dnf install -y httpd && dnf clean all -y

# Expose the default httpd port 80
EXPOSE 80

# Run the httpd
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
```

This time, when we build the image, we will get a 404 error produced by
the registry, which is unable to find an associated manifest for the
`sometag`{.inlineCode} tag:

``` {.programlisting .snippet-con}
$ buildah build -t custom_httpd .
STEP 1/4: FROM docker.io/library/fedora:sometag
Trying to pull docker.io/library/fedora:sometag...
error creating build container: initializing source docker://fedora:sometag: reading manifest sometag in docker.io/library/fedora: manifest unknown: manifest unknown
```

The missing tag will again generate an error that\'s easy to
troubleshoot, telling us that the manifest `sometag`{.inlineCode} cannot
be found.

**Solution**: Find []{#Chapter_11.xhtml#idx_1a015aea .index-entry
index-entry="builds:troubleshooting, from Dockerfiles"}a valid tag to be
used for the build process.[]{.sentence-end} Use
`skopeo`{.inlineCode}` list-tags`{.inlineCode} to find all the available
tags in a given repository, as shown in the
[]{#Chapter_11.xhtml#idx_4e9d38e2 .index-entry
index-entry="Dockerfiles:builds, triubleshooting from"}following
example:

``` {.programlisting .snippet-con}
$ skopeo list-tags docker://docker.io/library/fedora
{
    "Repository": "docker.io/library/fedora",
    "Tags": [
   ...omitted output...
        "39",
        "40",
        "branched",
        "heisenbug",
        "latest",
        "modular",
        "rawhide"
    ]
}    
```

Please also consider that, sometimes, the error caught from the
`FROM`{.inlineCode} instruction is caused by the attempt to access a
private registry without authentication.[]{.sentence-end} This is a very
common mistake and simply requires an authenticating step on the target
registry before any build action takes place.

In the next example, we have a Dockerfile that uses an image from a
generic private registry running using Docker Registry v2 APIs:

`Chapter11`{.inlineCode}`/`{.inlineCode}`FROM_auth_error`{.inlineCode}`/`{.inlineCode}`Dockerfile`{.inlineCode}

``` {.programlisting .snippet-code}
FROM local-registry.example.com/ubi8

# Update image and install httpd
RUN yum install -y httpd && yum clean all -y

# Expose the default httpd port 80
EXPOSE 80

# Run the httpd
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
```

Let\'s try to build the image and see what happens:

``` {.programlisting .snippet-con}
$ buildah build -t test3 .
STEP 1/4: FROM local-registry.example.com/ubi8
Trying to pull local-registry.example.com/ubi8:latest...
error creating build container: initializing source docker://local-registry.example.com/ubi8:latest: reading manifest latest in local-registry.example.com/ubi8: unauthorized: authentication required
```

In this use case, the []{#Chapter_11.xhtml#idx_3d4acff6 .index-entry
index-entry="builds:troubleshooting, from Dockerfiles"}error is very
clear.[]{.sentence-end} We are not authorized to pull
[]{#Chapter_11.xhtml#idx_67265065 .index-entry
index-entry="Dockerfiles:builds, triubleshooting from"}the image from
the target registry, and thus, we need to authenticate with a valid auth
token to access it.

**Solution**: Authenticate with
`podman`{.inlineCode}` `{.inlineCode}`login`{.inlineCode} or
`buildah`{.inlineCode}` `{.inlineCode}`login`{.inlineCode} to the
registry to retrieve the token or provide an authentication file with a
valid token.

So far, we have inspected errors generated by builds with
Dockerfiles.[]{.sentence-end} Let\'s now see the behavior of Buildah in
the case of errors when using its command-line instructions.

## Troubleshooting builds with Buildah-native commands {#Chapter_11.xhtml#h2_289 .heading-2}

When running Buildah []{#Chapter_11.xhtml#idx_eb346c5d .index-entry
index-entry="builds:troubleshooting, with Buildah-native commands"}commands,
it is a []{#Chapter_11.xhtml#idx_ff9c7f7e .index-entry
index-entry="Buildah-native commands:builds, troubleshooting"}common
practice to put them inside a shell script or a pipeline.

In this example, we will use Bash as the interpreter.[]{.sentence-end}
By default, Bash executes the script up to the end, regardless of
intermediate errors.[]{.sentence-end} This behavior can generate
unexpected errors if a Buildah instruction inside the script
fails.[]{.sentence-end} For this reason, the best practice is to add the
following command at the beginning of the script:

``` {.programlisting .snippet-con}
set -euo pipefail
```

The resulting configuration is a sort of safety net that blocks the
execution of the script as soon as we encounter an error and avoids
common mistakes, such as unset variables.

The `set`{.inlineCode} command is a Bash internal instruction that
configures the shell for script execution.[]{.sentence-end} The
`-e`{.inlineCode} option inside this instruction tells the shell to exit
immediately if a pipeline or a single command fails, and the
`-`{.inlineCode}`o`{.inlineCode}` `{.inlineCode}`pipefail`{.inlineCode}
option tells the shell to exit with the error code of the rightmost
command of a failing pipeline that produced a non-zero exit
code.[]{.sentence-end} The `-u`{.inlineCode} option tells the shell to
treat unset variables and parameters as an error during parameter
expansion.[]{.sentence-end} This keeps us safe from the missing
expansion of unset variables.

The next script embeds the logic of a simple build of an
`httpd`{.inlineCode} server on top of the Fedora image:

``` {.programlisting .snippet-con}
#!/bin/bash

set -euo pipefail
# Trying to pull a non-existing tag of Fedora official image
container=$(buildah from docker.io/library/fedora:non-existing-tag)
buildah run $container -- dnf install -y httpd; dnf clean all -y
buildah config --cmd "httpd -DFOREGROUND" $container
buildah config --port 80 $container
buildah commit $container custom-httpd
buildah tag custom-httpd registry.example.com/custom-httpd:v0.0.1
```

The image tag was []{#Chapter_11.xhtml#idx_23238e0d .index-entry
index-entry="builds:troubleshooting, with Buildah-native commands"}set
wrong on purpose.[]{.sentence-end} Let\'s see
the[]{#Chapter_11.xhtml#idx_d5ec26b8 .index-entry
index-entry="Buildah-native commands:builds, troubleshooting"} results
of the script execution:

``` {.programlisting .snippet-con}
$ ./custom-httpd.sh
Trying to pull docker.io/library/fedora:non-existing-tag...
initializing source docker://fedora:non-existing-tag: reading manifest non-existing-tag in docker.io/library/fedora: manifest unknown: manifest unknown
```

The build produces a
`manifest`{.inlineCode}` `{.inlineCode}`unknown`{.inlineCode} error,
just like the similar attempt with the Dockerfile.

From this output, we can also learn that Buildah (and Podman, which uses
Buildah libraries for its build implementation) produces the same
messages as a standard build with a Dockerfile/Containerfile, with the
only exception of not mentioning the build step, which is obvious since
we are running free commands inside a script.

**Solution**: Find a []{#Chapter_11.xhtml#idx_994222a2 .index-entry
index-entry="builds:troubleshooting, with Buildah-native commands"}valid
tag to be used for the build[]{#Chapter_11.xhtml#idx_0a5eb47d
.index-entry
index-entry="Buildah-native commands:builds, troubleshooting"}
process.[]{.sentence-end} Use
`skopeo`{.inlineCode}` `{.inlineCode}`list-tags`{.inlineCode} to find
all the available tags in a given repository.

In this section, we have learned how to analyze and troubleshoot build
errors, but what can we do when the errors happen at runtime inside the
container, and we do not have the proper tools for troubleshooting
inside the image? For this purpose, we have a native Linux tool that can
be considered the real Swiss Army knife of namespaces:
`nsenter`{.inlineCode}.

# Advanced troubleshooting with nsenter {#Chapter_11.xhtml#h1_290 .heading-1}

Let\'s start with a dramatic []{#Chapter_11.xhtml#idx_936209c3
.index-entry
index-entry="nsenter:for advanced troubleshooting"}sentence:
troubleshooting issues at runtime can sometimes be complex.

Also, understanding and troubleshooting runtime issues inside a
container implies an understanding of how containers work in
GNU/Linux.[]{.sentence-end} We explained these concepts in *Chapter*
*1*, *Introduction* *to* *Container* *Technology*.

Sometimes, troubleshooting can be very easy, and, as stated in the
previous sections, the usage of basic commands, such as
`podman`{.inlineCode}` `{.inlineCode}`logs`{.inlineCode},
`podman`{.inlineCode}` `{.inlineCode}`inspect`{.inlineCode}, and
`podman`{.inlineCode}` `{.inlineCode}`exec`{.inlineCode}, along with the
usage of tailored health checks, can help us to gain access to the
necessary information to complete our analysis successfully.

Images nowadays tend to be as small as possible.[]{.sentence-end} What
happens when we need more specialized troubleshooting tools, and they
are not available inside the image? You could think to execute a shell
process inside the container and install the missing tool, but sometimes
(and this is a growing security pattern), package managers are not
available inside container images, sometimes not even the
`curl`{.inlineCode} or `wget`{.inlineCode} commands!

We may feel a bit lost, but we must remember that containers are
processes executed within dedicated namespaces and
cgroups.[]{.sentence-end} What if we had a tool that could let us
execute inside one or more namespaces while keeping our access to the
host tools? That tool exists and is called `nsenter`{.inlineCode}
(access the manual page with
`man`{.inlineCode}` `{.inlineCode}`nsenter`{.inlineCode}).[]{.sentence-end}
It is not affiliated with any container engine or runtime and provides a
simple way to execute commands inside one or multiple namespaces
unshared for a process (the main container process).

Before diving into real[]{#Chapter_11.xhtml#idx_cc4e5574 .index-entry
index-entry="nsenter:for advanced troubleshooting"} examples, let\'s
discuss the main `nsenter`{.inlineCode} options and arguments by running
it with the `--help`{.inlineCode} option:

``` {.programlisting .snippet-con}
$ nsenter --help

Usage:
 nsenter [options] [<program> [<argument>...]]

Run a program with namespaces of other processes.

Options:
-a, --all              enter all namespaces
-t, --target <pid>     target process to get namespaces from
-m, --mount[=<file>]   enter mount namespace
-u, --uts[=<file>]     enter UTS namespace (hostname etc)
-i, --ipc[=<file>]     enter System V IPC namespace
-n, --net[=<file>]     enter network namespace
-p, --pid[=<file>]     enter pid namespace
-C, --cgroup[=<file>]  enter cgroup namespace
-U, --user[=<file>]    enter user namespace
-T, --time[=<file>]    enter time namespace
-S, --setuid <uid>     set uid in entered namespace
-G, --setgid <gid>     set gid in entered namespace
     --preserve-credentials do not touch uids or gids
-r, --root[=<dir>]     set the root directory
-w, --wd[=<dir>]       set the working directory
-F, --no-fork          do not fork before exec'ing <program>
-Z, --follow-context   set SELinux context according to --target PID

-h, --help             display this help
-V, --version          display version

For more details see nsenter(1).
```

From the output of this command, it is easy to spot that there are as
many options as the number of available namespaces.

Thanks to `nsenter`{.inlineCode}, we can capture the PID of the main
process of a container and then execute commands (including a shell)
inside the related namespaces.

To extract the container\'s main PID, we can use the following command:

``` {.programlisting .snippet-con}
$ podman inspect <Container_Name> --format '{{ .State.Pid }}'
```

The output can[]{#Chapter_11.xhtml#idx_5c3afbf3 .index-entry
index-entry="nsenter:for advanced troubleshooting"} be inserted inside a
variable for easier access:

``` {.programlisting .snippet-con}
$ CNT_PID=$(podman inspect <Container_Name> \
  --format '{{ .State.Pid }}')
```

 note
**Hint**

All namespaces associated with a process are represented inside the
`/`{.inlineCode}`proc`{.inlineCode}`/[`{.inlineCode}`pid`{.inlineCode}`]/ns`{.inlineCode}
directory.[]{.sentence-end} This directory contains a series of symbolic
links mapping to a namespace type and its corresponding inode
number.[]{.sentence-end} At its core, an inode is a data structure that
stores crucial information about a file or directory, while the inode
number is a unique integer that identifies that specific inode within a
given filesystem.

The following command shows the namespaces associated with the process
executed by the container:
`ls`{.inlineCode}` `{.inlineCode}`-`{.inlineCode}`al`{.inlineCode}` `{.inlineCode}`/`{.inlineCode}`proc`{.inlineCode}`/$`{.inlineCode}`CNT_PID`{.inlineCode}`/ns`{.inlineCode}.


We are going to learn how to use `nsenter`{.inlineCode} with a practical
example.[]{.sentence-end} In the next subsection, we will try to network
troubleshoot a database client application that returns an HTTP internal
server error without mentioning any useful information in the
application logs.

## Troubleshooting a database client with nsenter {#Chapter_11.xhtml#h2_291 .heading-2}

It is not uncommon to work []{#Chapter_11.xhtml#idx_6cb4e853
.index-entry
index-entry="database client:troubleshooting, with nsenter"}on alpha
applications that still do not have[]{#Chapter_11.xhtml#idx_2cefa652
.index-entry index-entry="nsenter:database client, troubleshooting"}
logging correctly implemented or that have poor handling of log
messages.

The following example is a web application that extracts fields from a
Postgres database and prints out a JSON object with all the
occurrences.[]{.sentence-end} The verbosity of the application logs has
been intentionally left to a minimum, and no connection or query errors
are produced.

Consider that, with this last example, we are simulating a real workflow
for an application developer trying to leverage a database connection
with the application just developed.

For the sake of space, we will not print the application source code in
the book; however, it is available at the following URL for inspection:
[[https://github.com/PacktPublishing/Podman-for-DevOps-Second-Edition/tree/main/Chapter11/students]{.url}](https://github.com/PacktPublishing/Podman-for-DevOps-Second-Edition/tree/main/Chapter11/students){style="text-decoration: none;"}.

The folder also contains a SQL script to populate a sample
database.[]{.sentence-end} The application is built using the following
Dockerfile:

`Chapter11`{.inlineCode}`/students/`{.inlineCode}`Dockerfile`{.inlineCode}

``` {.programlisting .snippet-code}
FROM docker.io/library/golang AS builder

# Copy files for build
RUN mkdir -p /go/src/students/models
COPY go.mod main.go /go/src/students
COPY models/main.go /go/src/students/models

# Set the working directory
WORKDIR /go/src/students

# Download dependencies
RUN go get -d -v ./...

# Install the package
RUN go build -v

# Runtime image
FROM registry.access.redhat.com/ubi9/ubi-minimal:latest as bin
COPY --from=builder /go/src/students /usr/local/bin
COPY entrypoint.sh /

EXPOSE 8080

ENTRYPOINT ["/entrypoint.sh"]
```

As usual, we are going to build the container with Buildah:

``` {.programlisting .snippet-con}
# buildah build -t students-image .
```

 note
**Important** **note**

While we are building and running this container in rootful mode to
explore traditional network namespace segregation, it is important to
distinguish this from how Podman handles rootless networking.

Even in rootless mode, containers remain fully segregated within their
own network namespaces.[]{.sentence-end} The key difference lies in the
driver.[]{.sentence-end} The current default, **pasta**, provides a
high-performance translation layer between the container\'s layer 2
interface and the []{#Chapter_11.xhtml#idx_056e8f71 .index-entry
index-entry="pasta"}host\'s native layer 4 sockets.[]{.sentence-end}
This allows for seamless network isolation and security without
requiring elevated privileges or complex IP configurations on the
host.[]{.sentence-end} We will explore the mechanics of pasta and other
advanced networking features in *[Chapter
12](#Chapter_12.xhtml#h1_295){.chapref}*.


The container []{#Chapter_11.xhtml#idx_50cbb5f6 .index-entry
index-entry="database client:troubleshooting, with nsenter"}accepts a
set of custom flags to define the[]{#Chapter_11.xhtml#idx_fb44db8c
.index-entry index-entry="nsenter:database client, troubleshooting"}
database, host, port, and credentials.[]{.sentence-end} To see the help,
simply run the following command:

``` {.programlisting .snippet-con}
# podman run students-image students -help
%!(EXTRA string=students)
-database string
      Default application database (default "students")
-host string     Default host running the database (default "localhost")
-password string      Default database password (default "password"
-port string    Default database port (default "5432")
-username string       Default database username (default "admin")
```

We have been informed that the database is running on host
`pghost.example.com`{.inlineCode} on port `5432`{.inlineCode}, with
username `students`{.inlineCode} and password
`Podman_R0cks`{.inlineCode}`#`{.inlineCode}.

The next command runs the `students`{.inlineCode} web application with
the custom arguments:

``` {.programlisting .snippet-con}
# podman run --rm -d -p 8080:8080 \
   --name students_app students-image \
   students -host pghost.example.com \
   -port 5432 \
   -username students \
   -password Podman_R0cks#
```

The container []{#Chapter_11.xhtml#idx_0cb5882a .index-entry
index-entry="nsenter:database client, troubleshooting"}starts
successfully, and the only log message
printed[]{#Chapter_11.xhtml#idx_31dd2889 .index-entry
index-entry="database client:troubleshooting, with nsenter"} is the
following:

``` {.programlisting .snippet-con}
# podman logs students_app
2021/12/27 21:51:31 Connecting to host pghost.example.com:5432, database students
```

We can finally test the application and see what happens when we run a
query:

``` {.programlisting .snippet-con}
$ curl localhost:8080/students
Internal Server Error
```

The application can take some time to answer, but after a while, it will
print an internal server error (`500`{.inlineCode}) HTTP
message.[]{.sentence-end} We will find the reason in the following
paragraphs.[]{.sentence-end} Logs are not useful since nothing other
than the first boot message is printed.[]{.sentence-end} Besides, the
container was built with the UBI minimal image, which has a small
footprint of pre-installed binaries and no utilities for
troubleshooting.[]{.sentence-end} We can use `nsenter`{.inlineCode} to
inspect the container behavior, especially from a networking point of
view, by attaching our current shell program to the container network
namespace while keeping access to our host binaries.

On a new shell, we can find out the main process PID and populate a
variable with its value (notice the `sudo`{.inlineCode} command to run
the inspection with elevated privileges for the running rootful
container):

``` {.programlisting .snippet-con}
$ CNT_PID=$(sudo podman inspect students_app --format '{{ .State.Pid }}')
```

The following example[]{#Chapter_11.xhtml#idx_17a3c908 .index-entry
index-entry="nsenter:database client, troubleshooting"} runs Bash in the
container network []{#Chapter_11.xhtml#idx_e04ca5b7 .index-entry
index-entry="database client:troubleshooting, with nsenter"}namespace,
while retaining all the other host namespaces (again, notice the
`sudo`{.inlineCode} command to run `nsenter`{.inlineCode} with elevated
privileges):

``` {.programlisting .snippet-con}
$ sudo nsenter -t $CNT_PID -n /bin/bash
```

 note
Important note

It is possible to run any host binary directly from
`nsenter`{.inlineCode}.[]{.sentence-end} A command such as the following
is perfectly legitimate:

``` {.programlisting .snippet-con-info}
$ sudo nsenter -t $CNT_PID -n ip addr show
```


To demonstrate that we are really executing a shell attached to the
container network namespace, we can launch the
`ip`{.inlineCode}` `{.inlineCode}`addr`{.inlineCode}` `{.inlineCode}`show`{.inlineCode}
command:

``` {.programlisting .snippet-con}
# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: tap0: <BROADCAST,UP,LOWER_UP> mtu 65520 qdisc fq_codel state UNKNOWN group default qlen 1000
    link/ether fa:0b:50:ed:9d:37 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.100/24 brd 10.0.2.255 scope global tap0
       valid_lft forever preferred_lft forever
    inet6 fe80::f80b:50ff:feed:9d37/64 scope link
       valid_lft forever preferred_lft forever
# ip route
default via 10.0.2.2 dev tap0
10.0.2.0/24 dev tap0 proto kernel scope link src 10.0.2.100
```

The first command,
`ip`{.inlineCode}` `{.inlineCode}`addr`{.inlineCode}` `{.inlineCode}`show`{.inlineCode},
prints the IP configuration, with a basic `tap0`{.inlineCode} interface
connected to the host and the loopback interface.

The second command,
`ip`{.inlineCode}` `{.inlineCode}`route`{.inlineCode}, shows the default
routing table inside the container network namespace.

We can take a first[]{#Chapter_11.xhtml#idx_d330c36c .index-entry
index-entry="nsenter:database client, troubleshooting"} look at the
active connections using []{#Chapter_11.xhtml#idx_10ac3b1e .index-entry
index-entry="database client:troubleshooting, with nsenter"}the
`ss`{.inlineCode} tool, already available on our Fedora host:

``` {.programlisting .snippet-con}
# ss -atunp
Netid State     Recv-Q Send-Q Local Address:Port  Peer Address:PortProcess
tcp   TIME-WAIT 0      0         10.0.2.100:50728   10.0.2.100:8080
tcp   LISTEN    0      128                *:8080             *:*    users:("("students",pid=402788,fd=3))
```

We immediately spot that there are no established connections between
the application and the database host, which tells us that the issue is
probably related to routing, firewall rules, or name resolution issues
that prevent us from reaching the host correctly.

The next step is to try to manually connect to the database with the
`psql`{.inlineCode} client tool, available from the rpm
`postgresql`{.inlineCode} package:

``` {.programlisting .snippet-con}
# psql -h pghost.example.com
psql: error: could not translate host name "pghost.example.com" to address: Name or service not known
```

This message is quite clear: the host is not resolved by the DNS service
and causes the application to fail.[]{.sentence-end} To finally confirm
it, we can run the `dig`{.inlineCode} command, which returns an
`NXDOMAIN`{.inlineCode} error, a typical message from a DNS server to
say that the domain cannot be resolved and does not exist:

``` {.programlisting .snippet-con}
# dig pghost.example.com

; <<>> DiG 9.16.23-RH <<>> pghost.example.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 40669
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;pghost.example.com.        IN    A

;; Query time: 0 msec
;; SERVER: 192.168.200.1#53(192.168.200.1)
;; WHEN: Mon Dec 27 23:26:47 CET 2021
;; MSG SIZE  rcvd: 47
```

After checking with the development team, we discovered that the
database name had a missing dash, so it was misspelled, and the correct
name was `pg-host.example.com`{.inlineCode}.[]{.sentence-end} We can now
fix the issue by running the container with the correct name.

We now expect to see []{#Chapter_11.xhtml#idx_0aa40190 .index-entry
index-entry="nsenter:database client, troubleshooting"}the correct
results when launching the query[]{#Chapter_11.xhtml#idx_ebd1db6d
.index-entry
index-entry="database client:troubleshooting, with nsenter"} again:

``` {.programlisting .snippet-con}
$ curl localhost:8080/students
{"Id":10149,"FirstName":"Frank","MiddleName":"Vincent","LastName":"Zappa","Class":"3A","Course":"Composition"}
```

In this example, we have focused on network namespace troubleshooting,
but it is possible to attach our current shell program to multiple
namespaces by simply adding the related flags.

We can also simulate
`podman`{.inlineCode}` `{.inlineCode}`exec`{.inlineCode} by running the
command with the `-`{.inlineCode}`a`{.inlineCode} option:

``` {.programlisting .snippet-con}
$ sudo nsenter -t $CNT_PID -a /bin/bash
```

This command[]{#Chapter_11.xhtml#idx_5d3a4085 .index-entry
index-entry="nsenter:database client, troubleshooting"} attaches the
process to all the unshared namespaces,
including[]{#Chapter_11.xhtml#idx_c16bcf29 .index-entry
index-entry="database client:troubleshooting, with nsenter"} the mount
namespace, thus giving the same filesystem tree view that is seen by
processes inside the container.

While `podman`{.inlineCode}` exec`{.inlineCode} is the standard way to
run a process inside an existing container, it has one major limitation:
it can only run binaries that already exist within the container\'s
image.

In a production environment, you are likely using *minimal* images, such
as Red Hat\'s `ubi`{.inlineCode}`-micro`{.inlineCode} or
`ubi`{.inlineCode}`-minimal`{.inlineCode}, to reduce the attack surface
and image size.[]{.sentence-end} These images often lack basic
troubleshooting tools such as `ip`{.inlineCode}, `ps`{.inlineCode},
`tcpdump`{.inlineCode}, or even a shell like
`bash`{.inlineCode}.[]{.sentence-end} If you try to use
`podman`{.inlineCode}` exec`{.inlineCode} to troubleshoot such a
container, you will be met with an error because the tool you need
simply isn\'t there.

# Summary {#Chapter_11.xhtml#h1_292 .heading-1}

In this chapter, we focused on container troubleshooting, trying to
provide a set of best practices and tools to find and fix issues inside
a container at build time or runtime.

We started by showing off some common use cases during container
execution and build stages, and their related solutions.

Afterward, we introduced the concept of health checks and illustrated
how to implement solid probes on containers to monitor their statuses,
while showing the architectural concepts behind them.

In the third section, we learned about a series of common error
scenarios related to builds and showed how to solve them quickly.

In the final section, we introduced the `nsenter`{.inlineCode} command
and simulated a web frontend application that needed network
troubleshooting to find out the cause of an internal server
error.[]{.sentence-end} Thanks to this example, we learned how to
conduct advanced troubleshooting inside the container namespaces.

Now that we\'ve done a deep dive into troubleshooting techniques for
containers, we are ready to move on to the next chapter, where we will
take an advanced look at networking for containers.

# Further reading {#Chapter_11.xhtml#h1_293 .heading-1}

-   Podman troubleshooting guidelines:
    [[https://github.com/containers/podman/blob/main/troubleshooting.md]{.url}](https://github.com/containers/podman/blob/main/troubleshooting.md){style="text-decoration: none;"}

# Get This Book's PDF Version and Exclusive Extras {#Chapter_11.xhtml#h1_294 .heading-1}

Scan the QR code (or go to
[[packtpub.com/unlock]{.url}](https://packtpub.com/unlock){style="text-decoration: none;"}).[]{.sentence-end}
Search for this book by name, confirm the edition, and then follow the
steps on the page.

![Image](images/B31467_11_1.png){style="width:25%;"}

![Image](images/B31467_11_2.png){style="width:25%;"}

*Note: Keep your invoice handy.[]{.sentence-end} Purchases made directly
from* *Packt* *don't require one.*


[]{#Chapter_12.xhtml}

 {.section .chapter-first}
