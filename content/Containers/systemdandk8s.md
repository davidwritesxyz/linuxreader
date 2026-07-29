# 14 {#Chapter_14.xhtml#h1_330 .chapterNumber}

# Interacting with systemd and Kubernetes {#Chapter_14.xhtml#h1_331 .chapterTitle}

In previous chapters, we learned how to initialize and manage
containers, starting with simple concepts and arriving at advanced
ones.[]{.sentence-end} Containers represent a key technology for
application development in the latest Linux **operating system**
(**OS**) releases.[]{.sentence-end} For this reason, containers are only
the starting point for advanced developers and system
administrators.[]{.sentence-end} Once this technology is widely adopted
in an enterprise company or a technical project, the next step will be
to integrate it with the base OS and with system orchestration
platforms.

In this chapter, we\'re going to cover the following main topics:

-   Setting up the prerequisites for the host OS
-   Integrating containers into systemd via Quadlets
-   Generating Kubernetes YAML resources
-   Running Kubernetes resource files in Podman
-   Testing the results in Kubernetes

# Technical requirements {#Chapter_14.xhtml#h1_332 .heading-1}

To complete this chapter, you will need a machine with a working Podman
installation.[]{.sentence-end} As we mentioned in *[Chapter
3](#Chapter_3.xhtml#h1_83){.chapref}*, *Running* *the* *First
Container*, all the examples in this book were executed on a Fedora 40
system or later but can be reproduced on your choice of OS.

Having a good understanding of the topics that were covered in *[Chapter
4](#Chapter_4.xhtml#h1_114){.chapref}*, *Managing Running Containers*,
*[Chapter 5](#Chapter_5.xhtml#h1_133){.chapref}*, *Implementing Storage
for the Container\'s Data*, and *[Chapter
9](#Chapter_9.xhtml#h1_219){.chapref}*, *Pushing Images to a Container
Registry*, will help you grasp the topics we\'ll cover regarding
advanced containers.

You should also have a good understanding of system administration and
Kubernetes container orchestration.

# Setting up the prerequisites for the host OS {#Chapter_14.xhtml#h1_333 .heading-1}

As we saw in *[Chapter 1](#Chapter_1.xhtml#h1_14){.chapref}*,
*Introduction to Container Technology*, containers were created to help
simplify and[]{#Chapter_14.xhtml#idx_e388e213 .index-entry
index-entry="host OS:prerequisites"} create system services that can be
distributed on standalone hosts.

In the following sections, we will learn how to run MariaDB and a Git
service in containers while managing those containers like any other
service -- that is, through systemd and the `systemctl`{.inlineCode}
command.

First, let\'s introduce []{#Chapter_14.xhtml#idx_cf32c7b0 .index-entry
index-entry="systemd"}systemd, a system and service manager for Linux
that runs as the first process on boot (as PID 1) and acts as an
`init`{.inlineCode} system that brings up and maintains userspace
services.[]{.sentence-end} Once a new user logs in to the host system,
separate instances are executed to start their services.

The systemd daemon starts services and ensures priority with a
dependency system between various entities called
**units**.[]{.sentence-end} There[]{#Chapter_14.xhtml#idx_16381ee8
.index-entry index-entry="units"} are 11 different types of units.

Fedora 40 and later have systemd enabled and running by
default.[]{.sentence-end} We can check whether it is running properly by
using the following command:

``` {.programlisting .snippet-con}
# systemctl is-system-running
running
```

In the following []{#Chapter_14.xhtml#idx_1dbe2636 .index-entry
index-entry="host OS:prerequisites"}sections, we are going to work with
system unit files of the `service`{.inlineCode} type.[]{.sentence-end}
We can check the current ones by running the following command:

``` {.programlisting .snippet-con}
# systemctl list-units --type=service | head
  UNIT                      LOAD   ACTIVE SUB     DESCRIPTION
  abrt-journal-core.service  loaded active running Creates ABRT problems from coredumpctl messages
  abrt-oops.service  loaded active running ABRT kernel log watcher
  abrt-xorg.service  loaded active running ABRT Xorg log watcher
  abrtd.service  loaded active running ABRT Automated Bug Reporting Tool
```

 note
**Important** **note**

The systemd service and its internals are more complex, so they cannot
be summarized in a few lines.[]{.sentence-end} For additional
information, please refer to the related Linux manual.


Now that we\'ve confirmed that the host is ready to handle the heavy
lifting, we can transition from basic system management to Quadlets, the
current industry standard for integrating containerized workloads
directly into the OS service life cycle.

# Integrating containers into systemd via Quadlets {#Chapter_14.xhtml#h1_334 .heading-1}

Historically, Podman []{#Chapter_14.xhtml#idx_6b5ecd75 .index-entry
index-entry="Quadlets:containers, integrating into systemd"}utilized the
`podman generate systemd`{.inlineCode} command to create unit
files.[]{.sentence-end} However, this command is now deprecated and
should no longer be used.[]{.sentence-end} The Podman team found that
static unit generation was brittle; as best practices for unit files
evolved with each release, users often failed to regenerate their files,
missing critical bug fixes and performance updates.

The current recommended solution is Quadlets.[]{.sentence-end} Quadlets
allow you to describe a container in a declarative format very similar
to standard systemd configuration files.[]{.sentence-end} Instead of you
managing a complex `.service`{.inlineCode} file, systemd uses a
generator to automatically create the necessary units at runtime.

The most efficient way to create Quadlet files is through a utility
called `podlet`{.inlineCode}.[]{.sentence-end} This tool takes standard
`podman run`{.inlineCode} commands and transforms them into the Quadlet
format.

We will create two system services based on containers to create a Git
repository.

For this example, we will []{#Chapter_14.xhtml#idx_a7051590 .index-entry
index-entry="Quadlets:containers, integrating into systemd"}leverage two
well-known open source projects:

-   **Gitea**: The[]{#Chapter_14.xhtml#idx_7384bb46 .index-entry
    index-entry="Gitea"} Git repository, which also offers a nice web
    interface for code management
-   **MariaDB**: The[]{#Chapter_14.xhtml#idx_0f83c39b .index-entry
    index-entry="MariaDB"} SQL database for holding the data that\'s
    produced by the Gitea service

Let\'s start with an example.[]{.sentence-end} First, we need to
generate a password for a user of our database:

``` {.programlisting .snippet-con}
# export MARIADB_PASSWORD=my-secret-pw
# podman secret create MARIADB_PASSWORD --env MARIADB_PASSWORD
cb5eab6a00d2b69635a314188
```

Here, we exported the environment variable with the secret password we
are going to use and then leveraged a useful secrets management command
that we have not introduced previously:
`podman secret create`{.inlineCode}.[]{.sentence-end} Unfortunately,
this command holds the secret in plain text, though this is good enough
for our purpose.[]{.sentence-end} Since we are running these containers
as root, these secrets are stored on the filesystem with root-only
permissions.

We can inspect the secret with the following commands:

``` {.programlisting .snippet-con}
# podman secret ls
ID                         NAME              DRIVER      CREATED         UPDATED
cb5eab6a00d2b69635a314188  MARIADB_PASSWORD  file        18 seconds ago  18 seconds ago

# podman secret inspect cb5eab6a00d2b69635a314188
[
    {
        "ID": "cb5eab6a00d2b69635a314188",
        "CreatedAt": "2025-08-07T15:18:01.250800245Z",
        "UpdatedAt": "2025-08-07T15:18:01.250800245Z",
        "Spec": {
            "Name": "MARIADB_PASSWORD",
            "Driver": {
                "Name": "file",
                "Options": {
                    "path": "/var/lib/containers/storage/secrets/filedriver"
                }
            },
            "Labels": {}
        }
    }
]

# cat /var/lib/containers/storage/secrets/filedriver/secretsdata.json
{
  "cb5eab6a00d2b69635a314188": "bXktc2VjcmV0LXB3"
}
# ls -l /var/lib/containers/storage/secrets/filedriver/secretsdata.json
-rw-------. 1 root root 53 Aug  7 15:18 /var/lib/containers/storage/secrets/filedriver/secretsdata.json
```

Here, we have asked []{#Chapter_14.xhtml#idx_7c77db30 .index-entry
index-entry="Quadlets:containers, integrating into systemd"}Podman to
list and inspect the secret we created previously and looked at the
underlying filesystem for the file holding the secrets.

The file holding the secrets is a file in JSON format and, as we
mentioned previously, is in plain text.[]{.sentence-end} The first
string is the secret ID, while the second string is the value
Base64-encoded.[]{.sentence-end} If we tried to decode it with the
`BASE64`{.inlineCode} algorithm, we would see that it represents the
password we just added -- that is, `my-secret-pw`{.inlineCode}.

Even though the password is stored in plain text, it is good enough for
our example because we are using the root user, and this filestore has
root-only permission, as we can verify with the last command of the
previous output.

Now, we can continue setting up the database container.[]{.sentence-end}
We will start with the database setup because it is a dependency on our
Git server.

We must create a local folder in the host system where we can store
container data:

``` {.programlisting .snippet-con}
# mkdir -p /opt/var/lib/mariadb
```

We can also look[]{#Chapter_14.xhtml#idx_ba0ca4d7 .index-entry
index-entry="Quadlets:containers, integrating into systemd"} at the
public documentation of the container image to find out the right volume
path and the various environment variables to use to start our
container:

``` {.programlisting .snippet-con}
# podman run -d --network host --name mariadb -v \
 /opt/var/lib/mariadb:/var/lib/mysql:Z -e \
 MARIADB_DATABASE=gitea -e MARIADB_USER=gitea -e \
 MARIADB_RANDOM_ROOT_PASSWORD=true \
--secret=MARIADB_PASSWORD,type=env docker.io/mariadb:latest
61ae055ef6512cb34c4b3fe1d8feafe6ec174a25547728873932f064921762d1
```

We are going to run and test the container standalone first to check
whether there are any errors; then, we will transform it into a system
service.

In the preceding Podman command, we did the following:

-   We ran the container in detached mode
-   We assigned it a name -- that is, `mariadb`{.inlineCode}
-   We exposed the host network for simplicity; of course, we could
    limit and filter this connectivity
-   We mapped the storage volume with the newly created local directory
    while also specifying the `:Z`{.inlineCode} option to correctly
    assign the SELinux labels
-   We defined the environment variables to use at runtime by the
    container\'s processes, also providing the password\'s secret with
    the `--secret`{.inlineCode} option
-   We used the container image name we want to use -- that is,
    `docker.io/mariadb:latest`{.inlineCode}

We can also check whether the container is up and running by using the
following command:

``` {.programlisting .snippet-con}
# podman ps
CONTAINER ID  IMAGE                             COMMAND     CREATED        STATUS        PORTS       NAMES
a37da3b39e5b  docker.io/library/mariadb:latest  mariadbd    9 minutes ago  Up 9 minutes              mariadb
```

The easiest way to create a []{#Chapter_14.xhtml#idx_06e31325
.index-entry
index-entry="Quadlets:containers, integrating into systemd"}Quadlet
configuration file is through a utility[]{#Chapter_14.xhtml#idx_e9ceba7b
.index-entry index-entry="podlet"} called **podlet**.[]{.sentence-end}
Let\'s install it on our system:

``` {.programlisting .snippet-con}
# dnf install -y podlet
Last metadata expiration check: 3:31:19 ago on Thu 07 Aug 2025 12:06:37 PM UTC.
Dependencies resolved.
=====================================================================================
 Package          Architecture     Version                   Repository         Size
=====================================================================================
Installing:
 podlet           x86_64           0.3.0-1.fc40              updates           2.1 M

Transaction Summary
=====================================================================================
Install  1 Package

Total download size: 2.1 M
Installed size: 6.6 M
Is this ok [y/N]: y
Downloading Packages:
podlet-0.3.0-1.fc40.x86_64.rpm                       3.4 MB/s | 2.1 MB     00:00   
-------------------------------------------------------------------------------------
Total                                                2.0 MB/s | 2.1 MB     00:01    
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                             1/1
  Installing       : podlet-0.3.0-1.fc40.x86_64                                  1/1
  Running scriptlet: podlet-0.3.0-1.fc40.x86_64                                  1/1

Installed:
  podlet-0.3.0-1.fc40.x86_64                                                        

Complete!
```

Now let\'s generate the[]{#Chapter_14.xhtml#idx_039bd798 .index-entry
index-entry="Quadlets:containers, integrating into systemd"} Quadlet for
our running `mariadb`{.inlineCode} service:

``` {.programlisting .snippet-con}
# podlet podman run -d --network host --name mariadb -v /opt/var/lib/mariadb:/var/lib/mysql:Z -e MARIADB_DATABASE=gitea -e MARIADB_USER=gitea -e MARIADB_RANDOM_ROOT_PASSWORD=true --secret=MARIADB_PASSWORD,type=env docker.io/mariadb:latest
# mariadb.container
[Container]
ContainerName=mariadb
Environment=MARIADB_DATABASE=gitea MARIADB_USER=gitea MARIADB_RANDOM_ROOT_PASSWORD=true
Image=docker.io/mariadb:latest
Network=host
Secret=MARIADB_PASSWORD,type=env
Volume=/opt/var/lib/mariadb:/var/lib/mysql:Z
```

As you can see, we are just putting before the previous
`podman run`{.inlineCode} command the `podlet`{.inlineCode} command;
everything will be managed directly by the utility!

Now, let\'s look at the Git service.[]{.sentence-end} First, we will
create the storage directory:

``` {.programlisting .snippet-con}
# mkdir -p /opt/var/lib/gitea/data
```

After that, we can[]{#Chapter_14.xhtml#idx_941dca44 .index-entry
index-entry="Quadlets:containers, integrating into systemd"} look at the
project documentation for any configuration that\'s needed for the Gitea
container image to be built correctly and complete the
`podman run`{.inlineCode} command:

``` {.programlisting .snippet-con}
# podman run -d --network host --name gitea \
-v /opt/var/lib/gitea/data:/data:Z \
docker.io/gitea/gitea:latest
fb91f23bccc318f90748ffd6ceebdf66ac5375d3c6ac78f267527a00b92a71f6
```

In the previous Podman command, we did the following:

-   We ran the container in detached mode
-   We assigned it a name -- that is, `gitea`{.inlineCode}
-   We exposed the host network for simplicity; of course, we can limit
    and filter this connectivity
-   We mapped the storage volume with the newly created local directory
    while specifying the `:Z`{.inlineCode} option to correctly assign
    the SELinux labels

Finally, we can check whether the service is running properly by
inspecting its logs:

``` {.programlisting .snippet-con}
# podman logs gitea
2025/08/07 15:56:34 cmd/web.go:261:runWeb() [I] Starting Gitea on PID: 2
2025/08/07 15:56:34 cmd/web.go:114:showWebStartupMessage() [I] Gitea version: 1.24.4 built with GNU Make 4.4.1, go1.24.5 : bindata, timetzdata, sqlite, sqlite_unlock_notify
2025/08/07 15:56:34 cmd/web.go:115:showWebStartupMessage() [I] * RunMode: prod
2025/08/07 15:56:34 cmd/web.go:116:showWebStartupMessage() [I] * AppPath: /usr/local/bin/gitea
2025/08/07 15:56:34 cmd/web.go:117:showWebStartupMessage() [I] * WorkPath: /var/lib/gitea
2025/08/07 15:56:34 cmd/web.go:118:showWebStartupMessage() [I] * CustomPath: /var/lib/gitea/custom
2025/08/07 15:56:34 cmd/web.go:119:showWebStartupMessage() [I] * ConfigFile: /etc/gitea/app.ini
2025/08/07 15:56:34 cmd/web.go:120:showWebStartupMessage() [I] Prepare to run install page
2025/08/07 15:56:35 cmd/web.go:323:listen() [I] Listen: http://0.0.0.0:3000
2025/08/07 15:56:35 cmd/web.go:327:listen() [I] AppURL(ROOT_URL): http://localhost:3000/
2025/08/07 15:56:35 modules/graceful/server.go:50:NewServer() [I] Starting new Web server: tcp:0.0.0.0:3000 on PID: 2
```

As we can see, the []{#Chapter_14.xhtml#idx_cde4344f .index-entry
index-entry="Quadlets:containers, integrating into systemd"}Gitea
service is listening on port `3000`{.inlineCode}.[]{.sentence-end}
Let\'s point our web browser to `http://localhost:3000`{.inlineCode} to
install it with the required configuration:

<figure class="mediaobject">
<img src="images/B31467_14_1.png"
style="width:528.0px; height:538.88px;" alt="Image 1" />
</figure>

Figure 14.1 -- Gitea service installation page

In the preceding screenshot, we[]{#Chapter_14.xhtml#idx_3b7109be
.index-entry
index-entry="Quadlets:containers, integrating into systemd"} defined the
database\'s type, address, username, and password to complete the
installation.[]{.sentence-end} Once done, we should be redirected to the
login page, as follows:

<figure class="mediaobject">
<img src="images/B31467_14_2.png"
style="width:528.0px; height:397.64588528678297px;" alt="Image 2" />
</figure>

Figure 14.2 -- Gitea service login page

Once the []{#Chapter_14.xhtml#idx_2f0573af .index-entry
index-entry="Quadlets:containers, integrating into systemd"}configuration
is complete, we can generate and add the Quadlet configuration files to
the right configuration path:

``` {.programlisting .snippet-con}
# podlet podman run -d --network host --name mariadb -v /opt/var/lib/mariadb:/var/lib/mysql:Z -e MARIADB_DATABASE=gitea -e MARIADB_USER=gitea -e MARIADB_RANDOM_ROOT_PASSWORD=true --secret=MARIADB_PASSWORD,type=env docker.io/mariadb:latest > /etc/containers/systemd/mariadb.container
# podlet podman run -d --network host --name gitea -v /opt/var/lib/gitea/data:/data:Z docker.io/gitea/gitea:latest-rootless
```

Then, we can even manually edit the Gitea Quadlet unit file by adding a
dependency order to the MariaDB service through the special
`Requires`{.inlineCode} instruction:

``` {.programlisting .snippet-con}
# cat /etc/containers/systemd/gitea.container
# gitea.container
[Unit]
Requires=mariadb.container
[Container]
ContainerName=gitea
Image=docker.io/gitea/gitea:latest-rootless
Network=host
Volume=/opt/var/lib/gitea/data:/data:Z
```

Thanks to []{#Chapter_14.xhtml#idx_b3b1cb43 .index-entry
index-entry="Quadlets:containers, integrating into systemd"}the
`Requires`{.inlineCode} instruction, systemd will start the MariaDB
service first, then the Gitea service.

Now, we can stop the containers by starting them through the systemd
units:

``` {.programlisting .snippet-con}
# podman stop mariadb gitea
mariadb
gitea
```

Don\'t worry about the data -- previously, we mapped both containers to
a dedicated storage volume that holds the data.

We need to let the systemd daemon know about the new unit files we just
added.[]{.sentence-end} So, first, we need to run the following command:

``` {.programlisting .snippet-con}
# systemctl daemon-reload
```

After that, we can start the services through systemd and check their
statuses:

``` {.programlisting .snippet-con}
# systemctl start mariadb
# systemctl status mariadb
# systemctl status mariadb
● mariadb.service
     Loaded: loaded (/etc/containers/systemd/mariadb.container; generated)
    Drop-In: /usr/lib/systemd/system/service.d
             └─10-timeout-abort.conf
     Active: active (running) since Thu 2025-08-07 16:11:15 UTC; 21s ago

...
# systemctl start gitea
# systemctl status gitea
● gitea.service
     Loaded: loaded (/etc/containers/systemd/gitea.container; generated)
    Drop-In: /usr/lib/systemd/system/service.d
             └─10-timeout-abort.conf
     Active: active (running) since Thu 2025-08-07 16:12:46 UTC; 4s ago
...
```

Finally, we can[]{#Chapter_14.xhtml#idx_a87e2a27 .index-entry
index-entry="Quadlets:containers, integrating into systemd"} enable the
service to start them when the OS boots; we just need to add the
following sections for both the `mariadb.container`{.inlineCode} and
`gitea.container`{.inlineCode} files:

``` {.programlisting .snippet-code}
[Install]
# Start by default on boot
WantedBy=multi-user.target default.target
```

With that, we have set up and enabled two containerized system services
on our host OS.[]{.sentence-end} This process is simple and could be
useful for leveraging the containers\' features and capabilities,
extending them to system services.

Once your `.container`{.inlineCode} files are placed in the systemd
directories, the `podman quadlet`{.inlineCode} command provides a suite
of tools for inspecting and managing them without having to manually
sift through the `/etc/`{.inlineCode} or `/run/`{.inlineCode}
directory.[]{.sentence-end} This command acts as an administrative
bridge, allowing you to verify how Podman and systemd are collaborating
to run your services.

The most []{#Chapter_14.xhtml#idx_4ce36be9 .index-entry
index-entry="Quadlets:containers, integrating into systemd"}common task
is verifying which Quadlets are currently recognized by the
system.[]{.sentence-end} By running
`podman quadlet -`{.inlineCode}`v`{.inlineCode}, you can see a verbose
list of all files that the systemd generator has processed.

Now, we are ready to move on to the next advanced topic, where we will
learn how to generate Kubernetes resources.

# Generating Kubernetes YAML resources {#Chapter_14.xhtml#h1_335 .heading-1}

Kubernetes has []{#Chapter_14.xhtml#idx_14c0b643 .index-entry
index-entry="Kubernetes YAML resources:generating"}become the de facto
standard for multi-node container orchestration.[]{.sentence-end}
Kubernetes clusters allow multiple Pods to be executed across nodes
according to scheduling policies that reflect the node\'s load, labels,
capabilities, or hardware resources (for example, GPUs).

We have already described the concept of a Pod -- a single execution
group of one or more containers that share common namespaces (network,
IPC, and, optionally, PID namespaces).[]{.sentence-end} In other words,
we can think of Pods as sandboxes for containers.[]{.sentence-end}
Containers inside a Pod are executed and thus started, stopped, or
paused simultaneously.

One of the most promising features that was introduced by Podman is the
capability to generate Kubernetes resources in YAML
format.[]{.sentence-end} Podman can read the configuration of containers
or Pods and generate a `Pod`{.inlineCode} resource that is compliant
with Kubernetes API specifications.

Along with Pods, we can generate
Service[]{#Chapter_14.xhtml#idx_4bd812f7 .index-entry
index-entry="PersistentVolumeClaim (PVC)"} and **PersistentVolumeClaim**
(**PVC**) resources as well, which reflect the configurations of the
port mappings and volumes that are mounted inside containers.

We can use the generated Kubernetes resources inside Podman itself as an
alternative to the Docker Compose stacks or apply them inside a
Kubernetes cluster to orchestrate the execution of simple Pods.

Kubernetes has many ways to orchestrate how workloads are executed:
Deployments, StatefulSets, DaemonSets, Jobs, and
CronJobs.[]{.sentence-end} In every case, Pods are their
workload-minimal execution units, and the orchestration logic changes
based on that specific behavior.[]{.sentence-end} This means that we can
take a Pod resource that\'s been generated by Podman and easily adapt it
to be orchestrated in a more complex object, such as a Deployment, which
manages replicas and version rollouts of our applications, or a
DaemonSet, which guarantees that a singleton Pod instance is created for
every cluster node.

Now, let\'s learn how to generate Kubernetes YAML resources with Podman.

## Generating basic Pod resources from running containers {#Chapter_14.xhtml#h2_336 .heading-2}

The basic []{#Chapter_14.xhtml#idx_9659029c .index-entry
index-entry="basic Pod resources:generating, from running containers"}command
to generate a Kubernetes resource from Podman is
`podman kube`{.inlineCode}` generate`{.inlineCode}, followed by various
options and arguments, as shown in the following code:

``` {.programlisting .snippet-con}
$ podman kube generate [options] {CONTAINER|POD|VOLUME}
```

We can apply this command to a running container, Pod, or existing
volume.[]{.sentence-end} The command also allows you to use the
`-s, --service`{.inlineCode} option to generate `Service`{.inlineCode}
resources and `-f, --filename`{.inlineCode} to export contents to a file
(the default is to standard output).

Let\'s start with a basic example of a `Pod`{.inlineCode} resource
that\'s been generated from a running container.[]{.sentence-end} First,
we will start a rootless Nginx container:

``` {.programlisting .snippet-con}
$ podman run -d \
  -p 8080:80 --name nginx \
  docker.io/library/nginx
```

When the container is created, we can generate our Kubernetes
`Pod`{.inlineCode} resource:

``` {.programlisting .snippet-con}
$ podman kube generate nginx
# Save the output of this file and use kubectl create -f to import
# it into Kubernetes.
#
# Created with podman-5.2.5
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2025-08-07T16:19:45Z"
  labels:
    app: nginx-pod
  name: nginx-pod
spec:
  containers:
  - args:
    - nginx
    - -g
    - daemon off;
    image: docker.io/library/nginx:latest
    name: nginx
    ports:
    - containerPort: 80
      hostPort: 8080
```

Let\'s describe the []{#Chapter_14.xhtml#idx_b049d65b .index-entry
index-entry="basic Pod resources:generating, from running containers"}generated
output.[]{.sentence-end} Every new Kubernetes resource is always
composed of at least four fields:

-   `apiVersion`{.inlineCode}: This field describes the API version
    schema of the resource.[]{.sentence-end} The `Pod`{.inlineCode}
    object belongs to the `v1`{.inlineCode} version of the
    `core`{.inlineCode} APIs of Kubernetes.
-   `kind`{.inlineCode}: This field defines the resource kind, which is
    `Pod`{.inlineCode} in our example.
-   `metadata`{.inlineCode}: This field is an object that holds a set of
    resource metadata that usually includes `name`{.inlineCode},
    `namespace`{.inlineCode}, `labels`{.inlineCode}, and
    `annotations`{.inlineCode}, along with additional dynamic metadata
    that\'s created at runtime, such as
    `creationTimestamp`{.inlineCode}, `resourceVersion`{.inlineCode}, or
    the resource\'s `uid`{.inlineCode}.
-   `spec`{.inlineCode}: This field holds resource specifications and
    varies among different resources.[]{.sentence-end} For example, a
    `Pod`{.inlineCode} resource will contain a list of containers, along
    with their startup arguments, volumes, ports, or security contexts.

All the information that\'s embedded inside a Pod resource is enough to
start the Pod inside a Kubernetes cluster.[]{.sentence-end} Along with
the fields described previously, a fifth `status`{.inlineCode} field is
dynamically created when the Pod is running to describe its execution
status.

From the generated output, we can notice an `args`{.inlineCode} list for
every container, along with their startup commands, arguments, and
options.

When you\'re generating a Pod from a container with mapped ports, the
following `ports`{.inlineCode} list is created inside the Pod resource:

``` {.programlisting .snippet-code}
ports:
    - containerPort: 80
      hostPort: 8080
```

This means that []{#Chapter_14.xhtml#idx_0e2bed52 .index-entry
index-entry="basic Pod resources:generating, from running containers"}port
`80`{.inlineCode} must be exposed to the container and port
`8080`{.inlineCode} must be exposed on the host running
it.[]{.sentence-end} This information will be used by Podman when we
create containers and Pods with the
`podman kube`{.inlineCode}` play`{.inlineCode} command, as we will see
in the next section.

We can apply the output of the
`podman kube`{.inlineCode}` generate`{.inlineCode} command directly to a
Kubernetes cluster or save it to a file.[]{.sentence-end} To save it to
a file, we can use the `-f`{.inlineCode} option:

``` {.programlisting .snippet-con}
$ podman kube generate nginx -f nginx-pod.yaml
```

To apply the []{#Chapter_14.xhtml#idx_734208f4 .index-entry
index-entry="kubectl"}generated output to a running Kubernetes cluster,
we can use the Kubernetes CLI tool, **kubectl**.[]{.sentence-end} The
`kubectl create`{.inlineCode} command applies a resource object inside
the cluster:

``` {.programlisting .snippet-con}
$ podman kube generate nginx | kubectl create -f -
```

The basic Pod generation command can be enriched by creating the related
Kubernetes services, as described in the next subsection.

## Generating Pods and Services from running containers {#Chapter_14.xhtml#h2_337 .heading-2}

Pods[]{#Chapter_14.xhtml#idx_86a4f572 .index-entry
index-entry="Pods:generating, from running containers"} running inside a
Kubernetes cluster obtain unique IP addresses on a software-defined
network that\'s managed by the default network plugin.

These IPs are not routed externally -- we can only reach the Pod\'s IP
address from within the cluster.[]{.sentence-end} However, we need a
layer to balance multiple replicas of the same Pods and provide a DNS
resolution for a single abstraction frontend.[]{.sentence-end} In other
words, our application must be able to query for a given service name
and receive a unique IP address that abstracts from the Pods\' IPs,
regardless of the number of replicas.

 note
**Important** **note**

Native, cluster-scoped DNS name resolution in
[]{#Chapter_14.xhtml#idx_de0cbcba .index-entry
index-entry="CoreDNS service"}Kubernetes is implemented with the
**CoreDNS** service, which is started when the cluster control plane is
bootstrapped.[]{.sentence-end} CoreDNS is delegated to resolve internal
requests and to forward ones for external names to authoritative DNS
servers outside the cluster.


The[]{#Chapter_14.xhtml#idx_da5c1b13 .index-entry index-entry="Service"}
resource that describes the abstraction in one or more Pods in
Kubernetes is called a **Service**.

For example, we can have three replicas of the Nginx Pod running inside
our cluster and expose them with a unique IP.[]{.sentence-end} It
belongs to a `ClusterIP`{.inlineCode} type, and its allocation is
dynamic when the[]{#Chapter_14.xhtml#idx_dda8deaa .index-entry
index-entry="Service:generating, from running containers"} service is
created.[]{.sentence-end} `ClusterIP`{.inlineCode} Services are the
default in Kubernetes, and their assigned IPs are only local to the
cluster.

We can also create `NodePort`{.inlineCode}-type Services that
[]{#Chapter_14.xhtml#idx_d34c75e3 .index-entry
index-entry="Network Address Translation (NAT)"}use **Network Address
Translation** (**NAT**) so that the service can be reached from the
external world.[]{.sentence-end} We can do this by mapping the service
VIP and port to a local port on the cluster worker nodes.

If we have a cluster running on an infrastructure that allows dynamic
load balancing (such as a public cloud provider), we can create
`LoadBalancer`{.inlineCode}-type Services and have the provider manage
ingress traffic load balancing for us.

Pods running inside a Kubernetes cluster obtain unique IP addresses on a
software-defined network managed by a network plugin.[]{.sentence-end}
These IPs are not routed externally; they are only reachable from within
the cluster.[]{.sentence-end} To expose these Pods and provide DNS
resolution or internal load balancing, Kubernetes utilizes a
`Service`{.inlineCode} resource.[]{.sentence-end} Podman allows you to
create Services along with Pods by adding the `-s`{.inlineCode} option
to the `podman kube`{.inlineCode}` generate`{.inlineCode}
command.[]{.sentence-end} This allows them to be potentially reused
inside a Kubernetes cluster.

We should also consider that in a production Kubernetes environment, we
may also find a `LoadBalancer`{.inlineCode} Service, which leverages
cloud provider infrastructure to distribute traffic.[]{.sentence-end}
Because Podman is a []{#Chapter_14.xhtml#idx_ce5d4894 .index-entry
index-entry="Service:generating, from running containers"}single-node
engine, it does not support the `LoadBalancer`{.inlineCode} type; load
balancing across multiple nodes does not apply to a standalone
deployment.[]{.sentence-end} The goal of the
`podman generate kube`{.inlineCode} command is to be a starting point,
so the user can add additional details to Podman\'s YAML and produce the
deployment that they really intended.

The following example is a variation of the previous one and generates
the Service resource along with the previously described Pod:

``` {.programlisting .snippet-con}
$ podman kube generate -s nginx
# Save the output of this file and use kubectl create -f to import
# it into Kubernetes.
#
# Created with podman-4.0.0-rc4
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2022-02-12T21:54:02Z"
  labels:
    app: nginxpod
  name: nginx_pod
spec:
  ports:
  - name: "80"
    nodePort: 30582
    port: 80
    targetPort: 80
  selector:
    app: nginxpod
  type: NodePort
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2022-02-12T21:54:02Z"
  labels:
    app: nginxpod
  name: nginx_pod
spec:
  containers:
  - args:
    - nginx
    - -g
    - daemon off;
    image: docker.io/library/nginx:latest
    name: nginx
    ports:
    - containerPort: 80
      hostPort: 8080
    securityContext:
      capabilities:
        drop:
        - CAP_MKNOD
        - CAP_NET_RAW
        - CAP_AUDIT_WRITE
```

The generated output[]{#Chapter_14.xhtml#idx_8a19df72 .index-entry
index-entry="Service:generating, from running containers"} contains,
along with the Pod resource, a Service resource that exposes the Nginx
Pod using a selector field.[]{.sentence-end} The selector matches all
the Pods with the `app: nginxpod`{.inlineCode} label.

When the service is created inside a Kubernetes cluster, an internal,
non-routed VIP is allocated for the service.[]{.sentence-end} Since this
is a `NodePort`{.inlineCode}-type service, a **Destination NAT**
(**DNAT**) rule is []{#Chapter_14.xhtml#idx_a63e608c .index-entry
index-entry="Destination NAT (DNAT) rule"}created to match incoming
traffic on all the cluster nodes on port `30582`{.inlineCode} and
forward it to the service IP.

By default, Podman generates `NodePort`{.inlineCode}-type
services.[]{.sentence-end} Whenever a container or Pod is decorated with
a port mapping, Podman populates the `ports`{.inlineCode} object with a
list of ports and their related `nodePort`{.inlineCode} mappings inside
the manifest.

In our use case, we created the Nginx container by mapping its port,
`80`{.inlineCode}, to port `8080`{.inlineCode} on the
host.[]{.sentence-end} Here, Podman generated a Service that maps the
container\'s port, `80`{.inlineCode}, to port `30582`{.inlineCode} on
the cluster nodes.

The value of []{#Chapter_14.xhtml#idx_6f3c59d5 .index-entry
index-entry="Service:generating, from running containers"}creating
Kubernetes services and Pods from Podman is the ability to port the
workload to a Kubernetes platform with little effort.

In many cases, we work with composite, multi-tier applications that need
to be exported and recreated together.[]{.sentence-end} Podman allows us
to export multiple containers into a single Kubernetes Pod object or to
create and export multiple Pods to gain more control over our
application.[]{.sentence-end} In the next two subsections, we will see
both cases applied to a WordPress application and try to find out what
the best approach is.

## Generating a composite application in a single Pod {#Chapter_14.xhtml#h2_338 .heading-2}

In this first scenario, we []{#Chapter_14.xhtml#idx_d4e6b0c1
.index-entry
index-entry="composite application:generating, in single Pod"}will
implement a multi-tier application in a single Pod.[]{.sentence-end} The
advantages of this approach are that we can leverage the Pod as a single
unit that will execute multiple containers and that resource sharing
across them is simplified.

We will launch two containers -- one for MySQL and one for WordPress --
and export them as a single Pod resource.[]{.sentence-end} We will learn
how to work around some minor adjustments to make it work seamlessly
later during run tests.

 note
**Important** **note**

The following examples have been created in a rootless context, but can
be seamlessly applied to rootful containers too.

A set of scripts that will be useful for launching the stacks and the
generated Kubernetes YAML files is available in this book\'s GitHub
repository at
[[https://github.com/PacktPublishing/Podman-for-DevOps-Second-Edition/tree/main/Chapter14/kube]{.url}](https://github.com/PacktPublishing/Podman-for-DevOps-Second-Edition/tree/main/Chapter14/kube){style="text-decoration: none;"}.


First, we must create two volumes that will be used later by the
WordPress and MySQL containers:

``` {.programlisting .snippet-con}
$ for vol in dbvol wpvol; do podman volume create $vol; done
```

Then, we must create an empty Pod named `wordpress-pod`{.inlineCode}
with the necessary predefined port mappings:

``` {.programlisting .snippet-con}
$ podman pod create --name wordpress-pod -p 8080:80
```

Now, we can populate our Pod by creating the WordPress and MySQL
containers.[]{.sentence-end} Let\'s begin with the MySQL container:

``` {.programlisting .snippet-con}
$ podman create \
  --pod wordpress-pod --name db \
  -v dbvol:/var/lib/mysql
  -e MYSQL_ROOT_PASSWORD=myrootpasswd \
  -e MYSQL_DATABASE=wordpress \
  -e MYSQL_USER=wordpress \
  -e MYSQL_PASSWORD=wordpress \
  docker.io/library/mysql
```

Now, we can []{#Chapter_14.xhtml#idx_044e2581 .index-entry
index-entry="composite application:generating, in single Pod"}create the
WordPress container:

``` {.programlisting .snippet-con}
$ podman create \
  --pod wordpress-pod --name wordpress \
  -v wpvol:/var/www/html
  -e WORDPRESS_DB_HOST=127.0.0.1 \
  -e WORDPRESS_DB_USER=wordpress \
  -e WORDPRESS_DB_PASSWORD=wordpress \
  -e WORDPRESS_DB_NAME=wordpress \
  docker.io/library/wordpress
```

Here, we can see that the `WORDPRESS_DB_HOST`{.inlineCode} variable has
been set to `127.0.0.1`{.inlineCode} (the address of the loopback
device) since the two containers are going to run in the same Pod and
share the same network namespace.[]{.sentence-end} For this reason, we
let the WordPress container know that the MySQL service is listening on
the same loopback device.

Finally, we can[]{#Chapter_14.xhtml#idx_ef114e74 .index-entry
index-entry="composite application:generating, in single Pod"} start the
Pod with the `podman pod start`{.inlineCode} command:

``` {.programlisting .snippet-con}
$ podman pod start wordpress-pod
```

We can inspect the running containers with `podman ps`{.inlineCode}:

``` {.programlisting .snippet-con}
$ podman ps
CONTAINER ID  IMAGE                                        COMMAND               CREATED            STATUS                PORTS                 NAMES
19bf706f0eb8  localhost/podman-pause:4.0.0-rc4-1643988335                        About an hour ago  Up About an hour ago  0.0.0.0:8080->80/tcp  0400f8770627-infra
f1da755a846c  docker.io/library/mysql:latest               mysqld                About an hour ago  Up About an hour ago  0.0.0.0:8080->80/tcp  db
1f28ef82d58f  docker.io/library/wordpress:latest           apache2-foregroun...  About an hour ago  Up About an hour ago  0.0.0.0:8080->80/tcp  wordpress
```

Now, we can point our browser to `http://localhost:8080`{.inlineCode}
and confirm the appearance of the WordPress setup dialog screen:

<figure class="mediaobject">
<img src="images/B31467_14_3.png"
style="width:304.1930680359435px; height:652.8px;" alt="Image 3" />
</figure>

Figure 14.3 -- WordPress setup dialog screen

 note
**Important** **note**

The Pod also started a third **infra** container that initializes the
Pod\'s network and the IPC namespaces of our example.[]{.sentence-end}
The image is built directly in the background on the host the first time
a Pod is created and executes a `catatonit`{.inlineCode} process, an
`init`{.inlineCode} micro container written in C that\'s designed to
handle system signals and zombie process reaping.

This behavior of the Pod\'s infra image is directly inherited from
Kubernetes\'s Pod life cycle.[]{.sentence-end} For additional details,
refer to the following link:
[[https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/]{.url}](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/){style="text-decoration: none;"}.


Now, we are []{#Chapter_14.xhtml#idx_da7944d9 .index-entry
index-entry="composite application:generating, in single Pod"}ready to
generate our Pod YAML manifest with the
`podman kube`{.inlineCode}` generate`{.inlineCode} command and save it
to a file for reuse:

``` {.programlisting .snippet-con}
$ podman kube generate wordpress-pod \
  -f wordpress-single-pod.yaml
```

The preceding command generates a file with the following content:

``` {.programlisting .snippet-code}
# Save the output of this file and use kubectl create -f to import
# it into Kubernetes.
#
# Created with podman-5.4.2
apiVersion: v1
kind: Pod
metadata:
 annotations:
  io.kubernetes.cri-o.SandboxID/wordpress: 6f7d22b98e403ef9804e5d141628cf5abbe03f64b3e133065ddd78c30ea6aea4
 creationTimestamp: "2026-02-13T20:41:28Z"
 labels:
  app: wordpress-pod
 name: wordpress-pod
spec:
 containers:
 - args:
  - apache2-foreground
  env:
  - name: WORDPRESS_DB_HOST
   value: 127.0.0.1
  - name: WORDPRESS_DB_PASSWORD
   value: wordpress
  - name: WORDPRESS_DB_NAME
   value: wordpress
  - name: WORDPRESS_DB_USER
   value: wordpress
  image: docker.io/library/wordpress:latest
  name: wordpress
  ports:
  - containerPort: 80
   hostPort: 8080
  volumeMounts:
  - mountPath: /var/www/html
   name: wpvol-pvc
 volumes:
 - name: wpvol-pvc
  persistentVolumeClaim:
   claimName: wpvol
```

Our YAML []{#Chapter_14.xhtml#idx_bac60f0e .index-entry
index-entry="composite application:generating, in single Pod"}file holds
a single Pod resource with two containers inside.[]{.sentence-end} Note
that the previously defined environment variables have been created
correctly inside our containers (when using Podman v4.0.0 or later).

Also, notice that the two container volumes have been mapped to PVC
objects.

PVCs are Kubernetes resources that are used to request (in other words,
claim) a storage volume resource that satisfies a specific capacity and
consumption modes.[]{.sentence-end} The attached storage volume resource
is called []{#Chapter_14.xhtml#idx_e17b370c .index-entry
index-entry="PersistentVolume (PV)"}a **PersistentVolume** (**PV**) and
can be created manually or automatically by a
`StorageClass`{.inlineCode} resource that leverages a storage driver
that\'s compliant with []{#Chapter_14.xhtml#idx_3790e382 .index-entry
index-entry="Container Storage Interface (CSI)"}the **Container Storage
Interface** (**CSI**).

When we create a PVC in Kubernetes, the default
`StorageClass`{.inlineCode} provisions a PV that satisfies our storage
requests, and the two resources are bound together.[]{.sentence-end}
This approach decouples the storage request from storage provisioning
and makes storage consumption in Kubernetes more portable.

When Podman generates Kubernetes YAML files, PVC resources are not
exported by default.[]{.sentence-end} However, we can also export the
PVC resources to recreate them in Kubernetes with the
`podman kube`{.inlineCode}` generate`{.inlineCode}` `{.inlineCode}`<`{.inlineCode}`VOLUME`{.inlineCode}`_NAME`{.inlineCode}`>`{.inlineCode}
command.

The following command exports the WordPress application, along with its
volume definitions, as a PVC:

``` {.programlisting .snippet-con}
$ podman kube generate wordpress-pod wpvol dbvol
```

The following is an []{#Chapter_14.xhtml#idx_00f5156e .index-entry
index-entry="composite application:generating, in single Pod"}example of
the `dbvol`{.inlineCode} volume translated into a PVC:

``` {.programlisting .snippet-code}
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    volume.podman.io/driver: local
  creationTimestamp: "2026-02-13T20:42:11Z "
  name: dbvol
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
status: {}
```

This approach has the advantage of providing the necessary PVC
definitions to recreate the whole application in a Kubernetes cluster,
but it is not necessary to recreate the volume resources in Podman: if
they\'re not available, an empty volume with the same name will be
created automatically.

To recreate all the resource dependencies in a Kubernetes cluster, we
can also export the application\'s `Service`{.inlineCode} resource.

The following command exports everything in our WordPress example,
including Pods, services, and volumes:

``` {.programlisting .snippet-con}
$ podman kube generate -s wordpress-pod wpvol dbvol
```

Before we move on, let\'s[]{#Chapter_14.xhtml#idx_ca742f72 .index-entry
index-entry="composite application:generating, in single Pod"} briefly
dig into the single-Pod approach logic that was described in this
subsection and look at its advantages and possible limitations.

One great advantage of executing all the containers in a single Pod is
the simpler networking configuration -- one network namespace is shared
by all the running containers.[]{.sentence-end} This also means that we
don\'t have to create a dedicated Podman network to let the containers
communicate with each other.

On the other hand, this approach does not reflect the common Kubernetes
pattern of executing Pods.[]{.sentence-end} In Kubernetes, we would
prefer to split the WordPress Pod and the MySQL Pod to manage them
independently and have different services associated with
them.[]{.sentence-end} More separation implies more control and the
chance to update independently.

In the next subsection, you\'ll learn how to replicate this approach and
generate multiple Pods for every application tier.

## Generating composite applications with multiple Pods {#Chapter_14.xhtml#h2_339 .heading-2}

One of the[]{#Chapter_14.xhtml#idx_9f39e8ef .index-entry
index-entry="composite applications:generating, with multiple Pods"}
features of Docker Compose is that you can create different independent
containers that communicate with each other using a service abstraction
concept that is decoupled from the container\'s execution.

The Podman community (and many of its users) believe that
standardization toward Kubernetes YAML manifests to describe complex
workloads is useful to get closer to the mainstream orchestration
solution.

For this reason, the approach we\'ll describe in this section can become
a full replacement for Docker Compose while providing Kubernetes
portability at the same time.[]{.sentence-end} First, we will learn how
to prepare an environment that can be used to generate the YAML
manifests.[]{.sentence-end} After that, we can get rid of the workloads
and only use the Kubernetes YAML to run our workloads.

The following example can be executed with rootless containers and
networks.

 note
**Important** **note**

Before continuing, make sure that the previous example\'s Pod and
containers have been completely removed, along with their volumes, to
prevent any issues with port assignment or WordPress content
initialization.[]{.sentence-end} Please refer to the commands in this
book\'s GitHub repository as a reference:
[[https://github.com/PacktPublishing/Podman-for-DevOps-Second-Edition/tree/main/AdditionalMaterial]{.url}](https://github.com/PacktPublishing/Podman-for-DevOps-Second-Edition/tree/main/AdditionalMaterial){style="text-decoration: none;"}.


First, we need to[]{#Chapter_14.xhtml#idx_73ab4cfb .index-entry
index-entry="composite applications:generating, with multiple Pods"}
create a network.[]{.sentence-end} We have chosen the name
`kubenet`{.inlineCode} to identify it easily and leave it with the
default configuration for the sake of our example:

``` {.programlisting .snippet-con}
$ podman network create kubenet
```

Once the network has been created, the two `dbvol`{.inlineCode} and
`wpvol`{.inlineCode} volumes must be created:

``` {.programlisting .snippet-con}
$ for vol in wpvol dbvol; do podman volume create $vol; done
```

We want to generate two distinct Pods -- one for each
container.[]{.sentence-end} First, we must create the MySQL Pod and its
related container:

``` {.programlisting .snippet-con}
$ podman pod create -p 3306:3306 \
  --network kubenet \
  --name mysql-pod
$ podman create --name db \
  --pod mysql-pod \
  -v dbvol:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=myrootpasswd\
  -e MYSQL_DATABASE=wordpress \
  -e MYSQL_USER=wordpress \
  -e MYSQL_PASSWORD=wordpress \
  docker.io/library/mysql
```

Notice the port []{#Chapter_14.xhtml#idx_e59d4ed2 .index-entry
index-entry="composite applications:generating, with multiple Pods"}mapping,
which we can use to access the MySQL service from a client and create
the correct port mapping later in the Kubernetes service.

Now, let\'s create the WordPress Pod and container:

``` {.programlisting .snippet-con}
$ podman pod create -p 8080:80 \
  --network kubenet \
  --name wordpress-pod
$ podman create --name wordpress \
  --pod wordpress-pod \
  -v wpvol:/var/www/html \
  -e WORDPRESS_DB_HOST=mysql-pod \
  -e WORDPRESS_DB_USER=wordpress \
  -e WORDPRESS_DB_PASSWORD=wordpress \
  -e WORDPRESS_DB_NAME=wordpress \
  docker.io/library/wordpress
```

There is a very important variable in the preceding command that can be
considered the key to this approach: `WORDPRESS_DB_HOST`{.inlineCode} is
populated with the `mysql-pod`{.inlineCode} string, which is the name
that\'s been given to the MySQL Pod.

In Podman, the Pod\'s name will act as the service name of the
application, and the DNS daemon associated with the network
(`aardvark-dns`{.inlineCode} since Podman 4) will directly resolve the
Pod name to the associated IP address.[]{.sentence-end} This is a key
feature that makes multi-Pod applications a perfect replacement for
Compose stacks.

Now, we can start the two Pods and have all the containers up and
running:

``` {.programlisting .snippet-con}
$ podman pod start mysql-pod &&
  podman pod start wordpress-pod
```

Once again, pointing our browsers to
`http://localhost:8080`{.inlineCode} should lead us to the WordPress
first setup page (if everything was set up correctly).

Now, we are ready to export our Kubernetes YAML
manifest.[]{.sentence-end} We can choose to simply export the two Pod
resources or create a full export that also includes services and
volumes.[]{.sentence-end} This is useful if you need to import to a
Kubernetes cluster.

Let\'s start with the []{#Chapter_14.xhtml#idx_06a9c93d .index-entry
index-entry="composite applications:generating, with multiple Pods"}basic
version:

``` {.programlisting .snippet-con}
$ podman kube generate \
  -f wordpress-multi-pod-basic.yaml \
  wordpress-pod \
  mysql-pod
```

The output of the preceding code will contain nothing but the two Pod
resources:

``` {.programlisting .snippet-code}
# Save the output of this file and use kubectl create -f to import
# it into Kubernetes.
#
# Created with podman-5.4.2
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2026-02-13T20:43:21Z"
  labels:
    app: wordpress-pod
  name: wordpress-pod
spec:
  containers:
  - args:
    - apache2-foreground
    env:
    - name: WORDPRESS_DB_NAME
      value: wordpress
    - name: WORDPRESS_DB_HOST
      value: mysql-pod
    - name: WORDPRESS_DB_PASSWORD
      value: wordpress
    - name: WORDPRESS_DB_USER
      value: wordpress
    image: docker.io/library/wordpress:latest
    name: wordpress
    ports:
    - containerPort: 80
      hostPort: 8080
    resources: {}
    securityContext:
      capabilities:
        drop:
        - CAP_MKNOD
        - CAP_NET_RAW
        - CAP_AUDIT_WRITE
    volumeMounts:
    - mountPath: /var/www/html
      name: wpvol-pvc
  restartPolicy: Never
  volumes:
  - name: wpvol-pvc
    persistentVolumeClaim:
      claimName: wpvol
status: {}
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2026-02-13T20:44:38Z"
  labels:
    app: mysql-pod
  name: mysql-pod
spec:
  containers:
  - args:
    - mysqld
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: myrootpasswd
    - name: MYSQL_DATABASE
      value: wordpress
    - name: MYSQL_USER
      value: wordpress
    - name: MYSQL_PASSWORD
      value: wordpress
    image: docker.io/library/mysql:latest
    name: db
    ports:
    - containerPort: 3306
      hostPort: 3306
    resources: {}
    securityContext:
      capabilities:
        drop:
        - CAP_MKNOD
        - CAP_NET_RAW
        - CAP_AUDIT_WRITE
    volumeMounts:
    - mountPath: /var/lib/mysql
      name: dbvol-pvc
  restartPolicy: Never
  volumes:
  - name: dbvol-pvc
    persistentVolumeClaim:
      claimName: dbvol
status: {}
```

The resulting file is also available in this book\'s GitHub repository:

[[https://github.com/PacktPublishing/Podman-for-DevOps-Second-Edition/blob/main/Chapter14/kube/wordpress-multi-pod-basic.yaml]{.url}](https://github.com/PacktPublishing/Podman-for-DevOps-Second-Edition/blob/main/Chapter14/kube/wordpress-multi-pod-basic.yaml){style="text-decoration: none;"}.

As we will see in the next section, this YAML file is enough to recreate
a fully working WordPress []{#Chapter_14.xhtml#idx_8580d9cb .index-entry
index-entry="composite applications:generating, with multiple Pods"}application
on Podman from scratch.[]{.sentence-end} We can persist and version it
on a source control repository such as Git for future reuse.

The following code exports the two Pod resources, along with the PVC and
Service resources:

``` {.programlisting .snippet-con}
$ podman kube generate -s \
  -f wordpress-multi-pod-full.yaml \
  wordpress-pod \
  mysql-pod \
  dbvol \
  wpvol
```

The output of this command is also available in this book\'s GitHub
repository:

[[https://github.com/PacktPublishing/Podman-for-DevOps-Second-Edition/blob/main/Chapter14/kube/wordpress-multi-pod-full.yaml]{.url}](https://github.com/PacktPublishing/Podman-for-DevOps-Second-Edition/blob/main/Chapter14/kube/wordpress-multi-pod-full.yaml){style="text-decoration: none;"}.

This full manifest is useful for importing and testing our application
on a Kubernetes cluster, where the Service and PVC resources are
necessary.

Now, we are ready to test our generated resources in Podman and learn
how to reproduce full stack deployments with simple operations.

# Running Kubernetes resource files in Podman {#Chapter_14.xhtml#h1_340 .heading-1}

Now []{#Chapter_14.xhtml#idx_872cb41b .index-entry
index-entry="Kubernetes resource files:running, in Podman"}that we\'ve
learned how to generate Kubernetes []{#Chapter_14.xhtml#idx_645deb0e
.index-entry
index-entry="Podman:Kubernetes resource files, running"}YAML files
containing the necessary resources to deploy our applications, we want
to test them in a real scenario.

For this book, we will use the WordPress application again, both in its
simple form with a single container and in its multi-Pod variation.

The following examples are also available in this book\'s GitHub
repository -- you can choose to use the resources that have been
generated from your labs or use the prepared manifests in this book\'s
repository.

 note
**Important** **note**

Don\'t forget to clean up all the previous workloads before testing the
creation of Kubernetes resources with Podman.


For all our examples, we will use the
`podman kube`{.inlineCode}` play`{.inlineCode} command.[]{.sentence-end}
It offers us an easy and intuitive interface for managing the execution
of complex stacks with a good degree of customization.

The first example will be based on the single-Pod manifest:

``` {.programlisting .snippet-con}
$ podman kube play wordpress-single-pod.yaml
```

The preceding command creates a Pod called `wordpress-pod`{.inlineCode}
that\'s composed of the two containers, along with the necessary
volumes.[]{.sentence-end} Let\'s inspect the results and see what
happened:

``` {.programlisting .snippet-con}
$ podman pod ps
POD ID        NAME           STATUS      CREATED         INFRA ID      # OF CONTAINERS
5f8ecfe66acd  wordpress-pod  Running     4 minutes ago  46b4bdfe6a08  3
```

We can also check the running containers.[]{.sentence-end} Here, we
expect to see the two WordPress and MySQL containers and the third
infra-related `podman-pause`{.inlineCode}:

``` {.programlisting .snippet-con}
$ podman ps
CONTAINER ID  IMAGE                                        COMMAND               CREATED         STATUS             PORTS                 NAMES
46b4bdfe6a08  localhost/podman-pause:4.0.0-rc4-1643988335                        4 minutes ago  Up 4 minutes ago  0.0.0.0:8080->80/tcp  5f8ecfe66acd-infra
ef88a5c8d1e5  docker.io/library/mysql:latest               mysqld                4 minutes ago  Up 4 minutes ago  0.0.0.0:8080->80/tcp  wordpress-pod-db
76c6b6328653  docker.io/library/wordpress:latest           apache2-foregroun...  4 minutes ago  Up 4 minutes ago  0.0.0.0:8080->80/tcp  wordpress-pod-wordpress
```

Finally, we can verify whether the `dbvol`{.inlineCode} and
`wpvol`{.inlineCode} volumes have been created:

``` {.programlisting .snippet-con}
$ podman volume ls
DRIVER      VOLUME NAME
local       dbvol
local       wpvol
```

Before we look[]{#Chapter_14.xhtml#idx_45da0658 .index-entry
index-entry="Kubernetes resource files:running, in Podman"} at the more
articulated (and interesting) example[]{#Chapter_14.xhtml#idx_e6bff69d
.index-entry index-entry="Podman:Kubernetes resource files, running"}
with the multi-Pod manifest, we must clean up the
environment.[]{.sentence-end} We can do this manually or by using the
`--down`{.inlineCode} option of the
`podman kube`{.inlineCode}` play`{.inlineCode} command, which
immediately stops and removes the running Pods:

``` {.programlisting .snippet-con}
$ podman kube play --down wordpress-single-pod.yaml
Pods stopped:
5f8ecfe66acd01b705f38cd175fad222890ab612bf572807082f30ab37fd0b88
Pods removed:
5f8ecfe66acd01b705f38cd175fad222890ab612bf572807082f30ab37fd0b88
```

 note
**Important** **note**

Volumes are not removed by default since it can be useful to keep them
if containers have already written data on them, and, of course,
deleting volumes could be a potential risk for losing important data, so
Podman is much more cautious there.[]{.sentence-end} To remove unused
volumes, use the `podman volume prune`{.inlineCode} command.


Now, let\'s run the multi-Pod example using the basic exported manifest:

``` {.programlisting .snippet-con}
$ podman kube play --network kubenet \
  wordpress-multi-pod-basic.yaml
```

Notice the[]{#Chapter_14.xhtml#idx_6d6a4202 .index-entry
index-entry="Kubernetes resource files:running, in Podman"} additional
`--network`{.inlineCode} argument, which is used to specify the network
that the Pods will be attached to.[]{.sentence-end} This is necessary
information since the Kubernetes YAML file contains no information about
Podman networks.[]{.sentence-end} Our Pods
[]{#Chapter_14.xhtml#idx_af2a8b0d .index-entry
index-entry="Podman:Kubernetes resource files, running"}will be executed
in rootless mode and attached to the rootless `kubenet`{.inlineCode}
network.

We can check that the two Pods have been created correctly by using the
following command:

``` {.programlisting .snippet-con}
$ podman pod ps
POD ID        NAME           STATUS      CREATED        INFRA ID      # OF CONTAINERS
c9d775da0379  mysql-pod      Running     8 minutes ago  71c93fa6080b  2
3b497cbaeebc  wordpress-pod  Running     8 minutes ago  0c52ee133f0f  2
```

Now, we can inspect the running containers.[]{.sentence-end} The strings
that are highlighted in the following code represent the main workload
to differentiate from the infra containers:

``` {.programlisting .snippet-con}
$ podman ps --format "{{.Image }} {{.Names}}"
localhost/podman-pause:5.4.2-1743552000 3b497cbaeebc-infra
docker.io/library/wordpress:latest wordpress-pod-wordpress
localhost/podman-pause:5.4.2-1743552000 c9d775da0379-infra
docker.io/library/mysql:latest mysql-pod-db
```

The `podman volume ls`{.inlineCode} command confirms the existence of
the two volumes:

``` {.programlisting .snippet-con}
$ podman volume ls
DRIVER      VOLUME NAME
local       dbvol
local       wpvol
```

Finally, let\'s inspect the DNS behavior.[]{.sentence-end} On Podman 4
and newer versions, the name resolution for custom networks is managed
by the `aardvark-dns`{.inlineCode} daemon, while on Podman 3, it is
managed by `dnsmasq`{.inlineCode}.[]{.sentence-end} Since we assume that
you\'re using Podman 5 for these examples, let\'s look at its DNS
configuration.[]{.sentence-end} For rootless networks, we can find the
managed records in the
`/run/user/`{.inlineCode}`<`{.inlineCode}`UID`{.inlineCode}`>`{.inlineCode}`/containers/networks/aardvark-dns/`{.inlineCode}`<`{.inlineCode}`NETWORK_NAME`{.inlineCode}`>`{.inlineCode}
file.

In our example, the []{#Chapter_14.xhtml#idx_b83e171b .index-entry
index-entry="Kubernetes resource files:running, in Podman"}configuration
for the `kubenet`{.inlineCode} network is
as[]{#Chapter_14.xhtml#idx_e4436794 .index-entry
index-entry="Podman:Kubernetes resource files, running"} follows:

``` {.programlisting .snippet-con}
$ cat /run/user/1000/containers/networks/aardvark-dns/kubenet
10.89.0.1
0c52ee133f0fec5084f25bd89ad8bd0f6af2fc46d696e2b8161864567b0a920b 10.89.0.4  wordpress-pod,0c52ee133f0f
71c93fa6080b6a3bfe1ebad3e164594c5fa7ea584e180113d2893eb67f6f3b56 10.89.0.5  mysql-pod,71c93fa6080b
```

The most amazing thing from this output is the confirmation that the
name resolution now works at the Pod level, not at the container
level.[]{.sentence-end} This is fair if we think that the Pod
initialized the namespaces, including the network
namespace.[]{.sentence-end} For this reason, we can treat the Pod name
in Podman as a service name.

Here, we demonstrated how the Kubernetes manifests that are generated
with Podman can become a great replacement for the Docker Compose
approach while being more portable.[]{.sentence-end} Now, let\'s learn
how to import our generated resources into a test Kubernetes cluster.

# Testing the results in Kubernetes {#Chapter_14.xhtml#h1_341 .heading-1}

In this section, we want to import the multi-Pod YAML file, which is
enriched with the Service and PVC configurations, on Kubernetes.

To provide a repeatable environment, we will
[]{#Chapter_14.xhtml#idx_4e587015 .index-entry
index-entry="minikube"}use **minikube** (with a lowercase m), a portable
solution, to create an all-in-one Kubernetes cluster as the local
infrastructure.

The minikube project aims to provide a local Kubernetes cluster on
Linux, Windows, and macOS.[]{.sentence-end} It uses host virtualization
to spin up a VM that runs the all-in-one cluster or containerization to
create a control plane that runs inside a container.[]{.sentence-end} It
also provides a large set of add-ons to extend cluster functionalities,
such as ingress controllers, service meshes, registries, and logging.

Another widely adopted []{#Chapter_14.xhtml#idx_41f8d9bb .index-entry
index-entry="Kubernetes in Docker (KinD) project"}alternative to
spinning up a local Kubernetes cluster is the **Kubernetes in Docker**
(**KinD**) project, which is not explored in this book.[]{.sentence-end}
KinD runs a Kubernetes control plane inside a container that\'s driven
by Docker or Podman.

To set up minikube, users need virtualization support (KVM, VirtualBox,
Hyper-V, Parallels, or VMware) or a container runtime such as Docker or
Podman.

For brevity, we will not cover the technical steps necessary to
configure the virtualization support for the different OSs; instead, we
will use a GNU/Linux distribution.

 note
**Important** **note**

If you already own a running Kubernetes cluster or want to set up one in
an alternative way, you can skip the next minikube configuration quick
start and go to the *Running generated resource files in Kubernetes*
subsection.


## Setting up minikube {#Chapter_14.xhtml#h2_342 .heading-2}

Run the following commands to []{#Chapter_14.xhtml#idx_8ddfda1c
.index-entry index-entry="minikube:setting up"}download and install the
latest `minikube`{.inlineCode} binary:

``` {.programlisting .snippet-con}
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

$ sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

You can choose to run minikube with a virtualization or containerization
driver.[]{.sentence-end} To run minikube as a VM on the KVM driver, you
must install the **Qemu/KVM** and **libvirt** packages.

On Fedora, run the []{#Chapter_14.xhtml#idx_8ce2b8a5 .index-entry
index-entry="minikube:setting up"}following command to install all the
mandatory and default packages[]{#Chapter_14.xhtml#idx_d1d38b68
.index-entry index-entry="libvirt package"} using the
`@virtualization`{.inlineCode} package group:

``` {.programlisting .snippet-con}
$ sudo dnf install @virtualization
```

Now, start and enable the `libvirtd`{.inlineCode} service:

``` {.programlisting .snippet-con}
$ sudo systemctl enable --now libvirtd
```

To grant the user running minikube the proper permissions, append it to
the `libvirt`{.inlineCode} supplementary group (this operation requires
a new login to load the new group):

``` {.programlisting .snippet-con}
$ sudo usermod -aG libvirt $(whoami)
```

The following command statically configures the `kvm2`{.inlineCode}
driver as the default:

``` {.programlisting .snippet-con}
$ minikube config set driver kvm2
```

When the preceding command is executed for the first time, minikube will
automatically download the proper `kvm2`{.inlineCode} driver binary
before starting the VM.

Alternatively, you can choose to run minikube as a containerized service
with Docker or Podman.[]{.sentence-end} Assuming Podman is already
installed, we just need to ensure that the user running minikube can run
passwordless `sudo`{.inlineCode}.[]{.sentence-end} This is necessary
since the Kubernetes cluster must run in a rootfull container, so
privilege escalation is necessary.[]{.sentence-end} To allow
passwordless privilege escalation for Podman, edit the
`/etc/sudoers`{.inlineCode} file with the following command:

``` {.programlisting .snippet-con}
$ sudo visudo
```

Once opened, add the following line to the end of the file to grant
passwordless escalation for the Podman binary and save
it.[]{.sentence-end} Remember to replace
`<`{.inlineCode}`username`{.inlineCode}`>`{.inlineCode} with your
user\'s name:

``` {.programlisting .snippet-con}
<username> ALL=(ALL) NOPASSWD: /usr/bin/podman
```

The following command statically configures the `podman`{.inlineCode}
driver as the default:

``` {.programlisting .snippet-con}
$ minikube config set driver podman
```

 note
**Important** **note**

If your host is a VM running on a hypervisor such as KVM, and Podman is
installed on the host, minikube will detect the environment and set up
the default driver as `podman`{.inlineCode} automatically.


To use minikube, users[]{#Chapter_14.xhtml#idx_9255f327 .index-entry
index-entry="minikube:setting up"} also need to install the Kubernetes
CLI tool, kubectl.[]{.sentence-end} The following commands download and
install the latest Linux release:

``` {.programlisting .snippet-con}
$ version=$(curl -L -s https://dl.k8s.io/release/stable.txt) curl -LO "https://dl.k8s.io/release/${version}/bin/linux/amd64/kubectl $ sudo install -o root -g root \
  -m 0755 kubectl \
  /usr/local/bin/kubectl
```

Now, we are ready to run our Kubernetes cluster with minikube.

## Starting minikube {#Chapter_14.xhtml#h2_343 .heading-2}

To []{#Chapter_14.xhtml#idx_b59f10a0 .index-entry
index-entry="minikube:starting"}start minikube as a VM, use the CRI-O
container runtime inside the Kubernetes cluster:

``` {.programlisting .snippet-con}
$ minikube start --driver=kvm2 --container-runtime=cri-o
```

The `--driver`{.inlineCode} option is not necessary if
`kvm2`{.inlineCode} has already been configured as the default driver
with the `minikube config set driver`{.inlineCode} command.

To start minikube with Podman, use the CRI-O container runtime inside
the cluster:

``` {.programlisting .snippet-con}
$ minikube start --driver=podman --container-runtime=cri-o
```

Again, the `--driver`{.inlineCode} option is not necessary if
`podman`{.inlineCode} has already been configured as the default driver
with the `minikube config set driver`{.inlineCode} command.

To ensure that the cluster has been created correctly, run the following
command with the `kubectl`{.inlineCode} CLI.[]{.sentence-end} All the
[]{#Chapter_14.xhtml#idx_c250b389 .index-entry
index-entry="minikube:starting"}Pods should have the
`Running`{.inlineCode} status:

``` {.programlisting .snippet-con}
$ kubectl get pods -A
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE
kube-system   coredns-64897985d-gqnrn            1/1     Running   0          19s
kube-system   etcd-minikube                      1/1     Running   0          27s
kube-system   kube-apiserver-minikube            1/1     Running   0          27s
kube-system   kube-controller-manager-minikube   1/1     Running   0          27s
kube-system   kube-proxy-sj7xn                   1/1     Running   0          20s
kube-system   kube-scheduler-minikube            1/1     Running   0          33s
kube-system   storage-provisioner                1/1     Running   0          30s
```

 note
**Important** **note**

If one or more containers still have the
`ContainerCreating`{.inlineCode} status, wait a little longer for the
images to be pulled.

Also, note that the output may differ slightly if you\'re running
minikube with a Podman driver.[]{.sentence-end} In that case, an
additional Pod named `kindnet`{.inlineCode} will be created to help
manage CNI networking inside the cluster.


With that, we have set everything up for a local Kubernetes environment
and are ready to test our generated manifests.

## Running generated resource files in Kubernetes {#Chapter_14.xhtml#h2_344 .heading-2}

In the *Generating composite applications* *with multiple Pods* section,
we learned how to export a manifest file
from[]{#Chapter_14.xhtml#idx_4ef9d141 .index-entry
index-entry="generated resource files:running, in Kubernetes"} Podman
that included the Pod[]{#Chapter_14.xhtml#idx_79ed857e .index-entry
index-entry="Kubernetes:generated resource files, running"} resources,
along with the Service and PVC resources.[]{.sentence-end} The need to
export this set of resources is related to the way Kubernetes handles
workloads, storage, and exposed services.

Kubernetes services are needed to provide a resolution mechanism, as
well as internal load balancing.[]{.sentence-end} In our example, the
`mysql-pod`{.inlineCode} Pod will be mapped to a homonymous
`mysql-pod`{.inlineCode} service.

PVCs are required to define a storage claim that starts provisioning PVs
for our Pods.[]{.sentence-end} In minikube, automated provisioning is
implemented by a local `StorageClass`{.inlineCode} named
`minikube-hostpath`{.inlineCode}; it creates local directories in the
VM/container filesystem that are later bind-mounted inside the Pods\'
containers.

We can roll out our WordPress stack by using the
`kubectl create`{.inlineCode} command:

``` {.programlisting .snippet-con}
$ kubectl create -f wordpress-multi-pod-full.yaml
```

If not specified, all the resources will be created in the
`default`{.inlineCode} Kubernetes namespace.[]{.sentence-end} Let\'s
wait for the Pods to reach the `Running`{.inlineCode} status and inspect
the results.

First, we can inspect the Pods and services that have been created:

``` {.programlisting .snippet-con}
$ kubectl get pods
NAME            READY   STATUS    RESTARTS   AGE
mysql-pod       1/1     Running   0          48m
wordpress-pod   1/1     Running   0          48m
$ kubectl get svc
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP          53m
mysql-pod       NodePort    10.108.34.77   <none>        3306:30284/TCP   52m
wordpress-pod   NodePort    10.96.63.142   <none>        80:30408/TCP     52m
```

Notice that the two `mysql-pod`{.inlineCode} and
`wordpress-pod`{.inlineCode} services have been created with the
`NodePort`{.inlineCode} type and mapped to port `30000`{.inlineCode} or
an upper range.[]{.sentence-end} We will use the `30408`{.inlineCode}
port to test the WordPress []{#Chapter_14.xhtml#idx_671113a2
.index-entry
index-entry="generated resource files:running, in Kubernetes"}frontend.

The Pods are mapped[]{#Chapter_14.xhtml#idx_1ace0b0e .index-entry
index-entry="Kubernetes:generated resource files, running"} by the
services using label-matching logic.[]{.sentence-end} If the labels that
have been defined in the service\'s `selector`{.inlineCode} field exist
in the Pod, it becomes an endpoint to the service
itself.[]{.sentence-end} Let\'s view the current endpoints in our
project:

``` {.programlisting .snippet-con}
$ kubectl get endpoints
NAME            ENDPOINTS         AGE
kubernetes      10.88.0.6:8443    84m
mysql-pod       10.244.0.5:3306   4m9s
wordpress-pod   10.244.0.6:80     4m9s
```

 note
**Important** **note**

The `kubernetes`{.inlineCode} service and its related endpoint provide
API access to internal workloads.[]{.sentence-end} However, it is not
part of this book\'s examples, so it can be ignored in this context.


Let\'s also inspect the claims and their related volumes:

``` {.programlisting .snippet-con}
$ kubectl get pvc
NAME    STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
dbvol   Bound    pvc-4d4a047b-bd20-4bef-879c-c3d80f96d712   1Gi        RWO            standard       54m
wpvol   Bound    pvc-accd7947-1499-44b5-bac8-9345da7edc23   1Gi        RWO            standard       54m

$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM           STORAGECLASS   REASON   AGE
pvc-4d4a047b-bd20-4bef-879c-c3d80f96d712   1Gi        RWO            Delete           Bound    default/dbvol   standard                60m
pvc-accd7947-1499-44b5-bac8-9345da7edc23   1Gi        RWO            Delete           Bound    default/wpvol   standard                60m
```

The two PVC resources have been created and bound to two dynamically
provisioned PVs.[]{.sentence-end} So long as the PVC objects exist, the
related PV will stay untouched, even if the Pods are destroyed and
recreated.

Now, the []{#Chapter_14.xhtml#idx_da35ec6a .index-entry
index-entry="generated resource files:running, in Kubernetes"}WordPress
application can be tested.[]{.sentence-end} By default,
minikube[]{#Chapter_14.xhtml#idx_106f77ba .index-entry
index-entry="Kubernetes:generated resource files, running"} does not
deploy an ingress controller (even though this can be enabled with the
`minikube addons enable ingress`{.inlineCode} command), so we will use
the simple `NodePort`{.inlineCode} service to test the functionalities
of our application.

The current minikube VM/container IP must be obtained to reach the
exposed `NodePort`{.inlineCode} service.[]{.sentence-end} Port
`30408`{.inlineCode}, which is associated with the
`wordpress-pod`{.inlineCode} service, listens to the IP address that\'s
produced by the following command:

``` {.programlisting .snippet-con}
$ minikube ip
10.88.0.6
```

Now, we can point our browser to `http://10.88.0.6:30408`{.inlineCode}
and see the WordPress first setup screen.

To remove the WordPress application and all its related content, use the
`kubectl delete`{.inlineCode} command in the YAML manifest file:

``` {.programlisting .snippet-con}
$ kubectl delete -f wordpress-multi-pod-full.yaml
```

This []{#Chapter_14.xhtml#idx_05f121b1 .index-entry
index-entry="generated resource files:running, in Kubernetes"}command
removes all the resources that have []{#Chapter_14.xhtml#idx_783e563a
.index-entry
index-entry="Kubernetes:generated resource files, running"}been defined
in the file, including the generated PVs.

# Summary {#Chapter_14.xhtml#h1_345 .heading-1}

With that, we have reached the end of this book about Podman and its
companion tools.

First, we learned how to generate systemd unit files and control
containerized workloads as systemd services, which allows us to, for
example, automate container execution at system startup.

After that, we learned how to generate Kubernetes YAML
resources.[]{.sentence-end} Starting with basic concepts and examples,
we learned how to generate complex application stacks using both
single-Pod and multiple-Pod approaches, and illustrated how the latter
can provide a great alternative (that is also Kubernetes-compliant) to
the Docker Compose methodology.

Finally, we tested our results on Podman and a local Kubernetes cluster
that had been created with minikube to show the great portability of
this approach.

In the next chapter, we explore how to manage containers, Kubernetes,
and AI workloads using Podman Desktop.

# Further reading {#Chapter_14.xhtml#h1_346 .heading-1}

To learn more about the topics that were covered in this chapter, take a
look at the following resources:

-   The Catatonit repository on GitHub:
    [[https://github.com/openSUSE/catatonit]{.url}](https://github.com/openSUSE/catatonit){style="text-decoration: none;"}
-   Kubernetes PersistentVolumes definition:
    [[https://kubernetes.io/docs/concepts/storage/persistent-volumes/]{.url}](https://kubernetes.io/docs/concepts/storage/persistent-volumes/){style="text-decoration: none;"}
-   The minikube project\'s home page:
    [[https://minikube.sigs.k8s.io/]{.url}](https://minikube.sigs.k8s.io/){style="text-decoration: none;"}
-   The KinD project\'s home page:
    [[https://kind.sigs.k8s.io/]{.url}](https://kind.sigs.k8s.io/){style="text-decoration: none;"}
-   Podman community links:
    [[https://podman.io/community/]{.url}](https://podman.io/community/){style="text-decoration: none;"}

# Join us on Discord {#Chapter_14.xhtml#h1_347 .heading-1}

For discussions around the book and to connect with your peers, join us
on Discord at
[[packt.link/discordcloud]{.url}](https://packt.link/discordcloud){style="text-decoration: none;"}
or scan the QR code below:

![Image](images/B31467_14_4.png){style="width:25%;"}


[]{#Chapter_15.xhtml}

 {.section .chapter-first}