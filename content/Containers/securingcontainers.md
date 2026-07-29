# 10 {#Chapter_10.xhtml#h1_248 .chapterNumber}

# Securing Containers {#Chapter_10.xhtml#h1_249 .chapterTitle}

Security is becoming the hottest topic of the current
times.[]{.sentence-end} Enterprises and companies all over the world are
making huge investments in security practices and tools that should help
protect their systems from internal or external attacks.

As we saw in *Chapter* *1*, *Introduction* *to* *Container*
*Technology*, containers and their host systems can be considered a
medium to execute and keep a target application
running.[]{.sentence-end} Security should be applied to all levels of
the service architecture, from the base infrastructure to the target
application code, all while passing through the virtualization or
containerization layer.

In this chapter, we will look at the best practices and tools that could
help improve the overall security of our containerization
layer.[]{.sentence-end} In particular, we\'re going to cover the
following main topics:

-   Running rootless containers with Podman
-   Avoiding running containers with UID 0
-   Signing our container images
-   Customizing Linux kernel capabilities
-   Understanding SELinux interaction with containers

# Technical requirements {#Chapter_10.xhtml#h1_250 .heading-1}

To complete this chapter\'s examples, you will need a machine with a
working Podman installation.[]{.sentence-end} As we mentioned in
*Chapter* *3*, *Running* *the* *First* *Container*, all the examples in
this book have been executed on a Fedora 40 system or later, but they
can be reproduced on your OS of choice.

Having a good understanding of the topics covered in *Chapter* *4*,
*Managing* *Running* *Containers*, *Chapter* *5*, *Implementing*
*Storage* *for* *the* *Container\'s* *Data*, and *Chapter* *9*,
*Pushing* *Images* *to* *a* *Container* *Registry*, will help you better
understand the container security topics discussed in this chapter.

# Running rootless containers with Podman {#Chapter_10.xhtml#h1_251 .heading-1}

As we briefly saw in *Chapter* *4,* *Managing* *Running* *Containers*,
it is possible for Podman to let standard users without administrative
privileges run containers on a Linux host.[]{.sentence-end} These
containers are often referred to []{#Chapter_10.xhtml#idx_1f891e40
.index-entry index-entry="rootless containers"}as **rootless**
**containers**.

Rootless containers have many advantages,
including[]{#Chapter_10.xhtml#idx_a10a7546 .index-entry
index-entry="rootless containers:advantages"} the following:

-   They create an additional security layer that could block attackers
    trying to get root privileges on the host, even if the container
    engine, runtime, or orchestrator has been
    compromised.[]{.sentence-end} This enforces against container escape
    vulnerability issues, potentially any flaw that allows a process in
    a container to escape the enforced security
    constraints.[]{.sentence-end} If it happens in a rootless container,
    the escaped process can only gain non-root user privileges.
-   They can allow many unprivileged users to run containers on the same
    host, making the most of high-performance computing environments.

Let\'s think about the approach that\'s used by any Linux system to
handle traditional process services.[]{.sentence-end} Usually, the
package maintainers tend to create a dedicated user for scheduling and
running the target process.[]{.sentence-end} If we try to install an
Apache web server on our favorite Linux distro through the default
package manager, then we can find out that the installed service will
run through a dedicated user named `apache`{.inlineCode}.

This approach has been the best practice for years because, from a
security perspective, allowing fewer privileges improves security.

Using the same approach but with a rootless
container[]{#Chapter_10.xhtml#idx_e60d7a57 .index-entry
index-entry="Podman"} allows us to run the container process without the
need for additional privilege escalation.[]{.sentence-end} Additionally,
Podman is daemonless, so it will just create a child process.

Running rootless containers in Podman is pretty straightforward and, as
we saw in the previous chapters, many of the examples in this book can
be run as standard unprivileged users.[]{.sentence-end} Now, let\'s
learn what\'s behind the execution of a rootless container.

## The Podman Swiss Army knife -- subuid and subgid {#Chapter_10.xhtml#h2_252 .heading-2}

Modern Linux distributions use a version of the
`shadow-utils`{.inlineCode} package that leverages two files:
`/etc/subuid`{.inlineCode} and
`/etc/subgid`{.inlineCode}.[]{.sentence-end} These
[]{#Chapter_10.xhtml#idx_99795dd1 .index-entry
index-entry="subuid"}files are used []{#Chapter_10.xhtml#idx_72946681
.index-entry index-entry="subgid"}to determine which UIDs and GIDs can
be used to map a user namespace.

The default allocation for every user is `65536`{.inlineCode} UIDs and
`65536`{.inlineCode} GIDs.

We can run the following simple command to check how the
`subuid`{.inlineCode} and `subgid`{.inlineCode} allocation works in
rootless containers:

``` {.programlisting .snippet-con}
$ id
uid=1000(alex) gid=1000(alex) groups=1000(alex),10(wheel)
$ podman run alpine cat /proc/self/uid_map /proc/self/gid_map
Resolved "alpine" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull docker.io/library/alpine:latest...
Getting image source signatures
Copying blob 59bf1c3509f3 done
Copying config c059bfaa84 done
Writing manifest to image destination
Storing signatures
         0       1000          1
         1     100000      65536
         0       1000          1
         1     100000      65536
```

As we can see, both files indicate that they start mapping UID and GID 0
with the current user\'s UID/GID that
we[]{#Chapter_10.xhtml#idx_9062453b .index-entry index-entry="subuid"}
just ran the container with; that is,
`1000`{.inlineCode}.[]{.sentence-end} After that, it maps UID and GID 1,
starting []{#Chapter_10.xhtml#idx_eb0c0a4d .index-entry
index-entry="subgid"}from `100000`{.inlineCode} and arriving at
`165536`{.inlineCode}.[]{.sentence-end} This is calculated by summing
the starting point, `100000`{.inlineCode}, and the default range,
`65536`{.inlineCode}.

For a comparison, let\'s see the same example on the host:

``` {.programlisting .snippet-con}
$ id
uid=1000(alex) gid=1000(alex) groups=1000(alex),10(wheel)
$ cat /proc/self/uid_map /proc/self/gid_map
         0          0 4294967295
         0          0 4294967295
```

Because it starts at `0`{.inlineCode} for both sides and covers the full
range of 32-bit UID available on Linux, `4294967295`{.inlineCode}, it
means UID/GID 1000 on the host system is seen as UID/GID 1000 by the
process.[]{.sentence-end} There is no translation happening.

Using rootless containers is not the only best practice we can implement
for our container environments.[]{.sentence-end} In the next section,
we\'ll learn why we shouldn\'t run a container with UID
`0`{.inlineCode}.

# Avoiding running containers with UID 0 {#Chapter_10.xhtml#h1_253 .heading-1}

Container runtimes can be[]{#Chapter_10.xhtml#idx_86dc9fb1 .index-entry
index-entry="running containers:avoiding, with UID 0"} instructed to run
processes inside []{#Chapter_10.xhtml#idx_d5801c0c .index-entry
index-entry="UID 0:running containers, avoiding"}a container with a user
ID different from the one that initially created the
container.[]{.sentence-end} Running the processes inside the container
as a non-root user can be helpful for security
purposes.[]{.sentence-end} While container runtimes use Linux namespaces
to isolate processes, a vulnerability in the container engine or the
kernel could allow a root user (UID `0`{.inlineCode} in the container)
to *break out* of the container and gain the same privileges as the host
root user (UID `0`{.inlineCode} in the host).[]{.sentence-end} Using an
unprivileged user (with a non-0 UID) in a container can limit the
privilege escalation and thus the attack surface inside and outside the
container.

By default, a Dockerfile and Containerfile may set the default user as
root (that is, `UID=0`{.inlineCode}).[]{.sentence-end} To avoid this, we
can leverage the `USER`{.inlineCode} instruction in those build files --
for example, `USER`{.inlineCode}` `{.inlineCode}`1001`{.inlineCode} --
to instruct Buildah or other container build tools to build and run the
container image using that particular user (with UID
`1001`{.inlineCode}).[]{.sentence-end} As explained before, a potential
container breakout from the container will allow, in the worst case,
access to the files and resources of the host user with the same UID,
`1001`{.inlineCode}.

If we want to force a specific UID, we need to adjust the permissions of
any file, folder, or mount we plan to use with our running containers.

Now, let\'s learn how to adapt an existing image so that it can be run
with a standard user.

We can leverage some prebuilt images on Docker Hub or pick one of the
official nginx container images.[]{.sentence-end} First, we need to
create a basic `nginx`{.inlineCode} configuration file:

``` {.programlisting .snippet-con}
$ cat hello-podman.conf
server {
    listen 80;

    location / {
        default_type text/plain;
        expires -1;
        return 200 'Hello Podman user!\nServer address: $server_addr:$server_port\n';
    }
}
```

The `nginx`{.inlineCode} configuration []{#Chapter_10.xhtml#idx_bb9fca16
.index-entry index-entry="running containers:avoiding, with UID 0"}file
is really simple: we define the listening
[]{#Chapter_10.xhtml#idx_4b51dd61 .index-entry
index-entry="UID 0:running containers, avoiding"}port
(`80`{.inlineCode}) and the content message to return once a request
arrives on the server.

Then, we can create a simple Dockerfile to leverage one of the official
nginx container images:

``` {.programlisting .snippet-con}
$ cat Dockerfile
FROM docker.io/library/nginx:mainline-alpine
RUN rm /etc/nginx/conf.d/*
ADD hello-podman.conf /etc/nginx/conf.d/
```

The Dockerfile contains three instructions:

-   `FROM`{.inlineCode}: For selecting the official nginx image
-   `RUN`{.inlineCode}: For cleaning the configuration directory of any
    default config example
-   `ADD`{.inlineCode}: For copying the configuration file that we just
    created

Now, let\'s build the[]{#Chapter_10.xhtml#idx_f66908fc .index-entry
index-entry="running containers:avoiding, with UID 0"}
container[]{#Chapter_10.xhtml#idx_635ad8e5 .index-entry
index-entry="UID 0:running containers, avoiding"} image with Buildah:

``` {.programlisting .snippet-con}
$ buildah bud -t nginx-root:latest -f .
STEP 1/3: FROM docker.io/library/nginx:mainline-alpine
STEP 2/3: RUN rm /etc/nginx/conf.d/*
STEP 3/3: ADD hello-podman.conf /etc/nginx/conf.d/
COMMIT nginx-root:latest
Getting image source signatures
Copying blob 8d3ac3489996 done
...
Copying config 21c5f7d8d7 done
Writing manifest to image destination
Storing signatures
--> 21c5f7d8d70
Successfully tagged localhost/nginx-root:latest
21c5f7d8d709e7cfdf764a14fd6e95fb4611b2cde52b57aa46d43262a6489f41
```

Once you\'ve built the image, name it
`nginx-root`{.inlineCode}.[]{.sentence-end} Now, we are ready to run our
container:

``` {.programlisting .snippet-con}
$ podman run --name myrootnginx -p 127.0.0.1::80 -d nginx-root
364ec7f5979a5059ba841715484b7238db3313c78c5c577629364aa46b6d9bdc
```

Here, we used the `–p`{.inlineCode} option to publish the port and make
it reachable from the host.[]{.sentence-end} Let\'s find out which local
port has been chosen, randomly, in the host system:

``` {.programlisting .snippet-con}
$ podman port myrootnginx 80
127.0.0.1:38029
```

Finally, let\'s call our containerized web server:

``` {.programlisting .snippet-con}
$ curl localhost:38029
Hello Podman user!
Server address: 10.0.2.100:80
```

The container is finally running, but what user is our container using?
Let\'s find out:

``` {.programlisting .snippet-con}
$ podman ps | grep root
364ec7f5979a  localhost/nginx-root:latest  nginx -g daemon o...  55 minutes ago  Up 55 minutes ago  0.0.0.0:38029->80/tcp      myrootnginx
$ podman exec 364ec7f5979a id
uid=0(root) gid=0(root)
```

As expected, the []{#Chapter_10.xhtml#idx_7efb90c0 .index-entry
index-entry="running containers:avoiding, with UID 0"}container is
running as root!

Now, let\'s make a few[]{#Chapter_10.xhtml#idx_3aef7dc5 .index-entry
index-entry="UID 0:running containers, avoiding"} edits to change the
user.[]{.sentence-end} First, we need to change the listening port in
the nginx server configuration:

``` {.programlisting .snippet-con}
$ cat hello-podman.conf
server {
    listen 8080;

    location / {
        default_type text/plain;
        expires -1;
        return 200 'Hello Podman user!\nServer address: $server_addr:$server_port\n';
    }
}
```

Here, we replaced the listening port (`80`{.inlineCode}) with
`8080`{.inlineCode}; we cannot use a port that\'s below
`1024`{.inlineCode} with unprivileged users.

Then, we need to edit our Dockerfile:

``` {.programlisting .snippet-con}
$ cat Dockerfile
FROM docker.io/library/nginx:mainline-alpine
RUN rm /etc/nginx/conf.d/*
ADD hello-podman.conf /etc/nginx/conf.d/

RUN chmod -R a+w /var/cache/nginx/ \
        && touch /var/run/nginx.pid \
        && chmod a+w /var/run/nginx.pid
EXPOSE 8080
USER nginx
```

As you can see, we fixed the permissions for the main file and folder on
the nginx server, exposed the new `8080`{.inlineCode} port, and set the
default user to an nginx one.

Now, we are []{#Chapter_10.xhtml#idx_61141b46 .index-entry
index-entry="running containers:avoiding, with UID 0"}ready to build a
brand-new container image.[]{.sentence-end} Let\'s
[]{#Chapter_10.xhtml#idx_8c127501 .index-entry
index-entry="UID 0:running containers, avoiding"}call it
`nginx-user`{.inlineCode}:

``` {.programlisting .snippet-con}
$ buildah bud -t nginx-user:latest -f .
STEP 1/6: FROM docker.io/library/nginx:mainline-alpine
STEP 2/6: RUN rm /etc/nginx/conf.d/*
STEP 3/6: ADD hello-podman.conf /etc/nginx/conf.d/
STEP 4/6: RUN chmod -R a+w /var/cache/nginx/         && touch /var/run/nginx.pid         && chmod a+w /var/run/nginx.pid
STEP 5/6: EXPOSE 8080
STEP 6/6: USER nginx
COMMIT nginx-user:latest
Getting image source signatures
Copying blob 8d3ac3489996 done
...
Copying config 7628852470 done
Writing manifest to image destination
Storing signatures
--> 76288524704
Successfully tagged localhost/nginx-user:latest
762885247041fd233c7b66029020c4da8e1e254288e1443b356cbee4d73adf3e
```

Now, we can run the container:

``` {.programlisting .snippet-con}
$ podman run --name myusernginx -p 127.0.0.1::8080 -d nginx-user
299e0fb727f339d87dd7ea67eac419905b10e36181dc1ca7e35dc7d0a9316243
```

Find the associated random host port and check whether the web server is
working:

``` {.programlisting .snippet-con}
$ podman port myusernginx 8080
127.0.0.1:42209
$ curl 127.0.0.1:42209
Hello Podman user!
Server address: 10.0.2.100:8080
```

Finally, let\'s[]{#Chapter_10.xhtml#idx_1b6c0b12 .index-entry
index-entry="running containers:avoiding, with UID 0"} see whether we
changed the user that\'s running the []{#Chapter_10.xhtml#idx_ef44ec62
.index-entry index-entry="UID 0:running containers, avoiding"}target
process in our container:

``` {.programlisting .snippet-con}
$ podman ps | grep user
299e0fb727f3  localhost/nginx-user:latest  nginx -g daemon o...  38 minutes ago  Up 38 minutes ago  127.0.0.1:42209->8080/tcp  myusernginx
$ podman exec 299e0fb727f3 id
uid=101(nginx) gid=101(nginx) groups=101(nginx)
```

As you can see, our container is running as an unprivileged user, which
is what we wanted.

If you want to look at a ready-to-use example of this, please go to this
book\'s GitHub repository:
[[https://github.com/PacktPublishing/Podman-for-DevOps]{.url}](https://github.com/PacktPublishing/Podman-for-DevOps){style="text-decoration: none;"}.

Unfortunately, security is not all about permissions and users -- we
also need to take care of the base image and its source and check
container image signatures.[]{.sentence-end} We\'ll learn about this in
the next section.

# Signing our container images {#Chapter_10.xhtml#h1_254 .heading-1}

When we\'re dealing with[]{#Chapter_10.xhtml#idx_e664477f .index-entry
index-entry="container images:signing"} images that have been pulled
from external registries, we will have some security concerns related to
the potential attack tactics that have been conducted on the containers
(see \[*1*\] in the *Further* *reading* section), especially
masquerading techniques, which help the attacker manipulate image
components to make them appear legitimate.[]{.sentence-end} This could
also happen due[]{#Chapter_10.xhtml#idx_3d754214 .index-entry
index-entry="man-in-the-middle (MITM) attack"} to a
**man-in-the-middle** (**MITM**) attack being conducted by an attacker
over the wire.

To prevent certain kinds of attacks while you\'re managing containers,
the best solution is to use a detached image signature to trust the
image provider and guarantee its reliability.

When an image is pulled, Podman can verify the validity of the
signatures and reject images without valid
[]{#Chapter_10.xhtml#idx_055dbd15 .index-entry
index-entry="container images:signing"}signatures.

Now, let\'s learn how to implement a basic image signature workflow.

## Signing images with Sigstore and Podman {#Chapter_10.xhtml#h2_255 .heading-2}

In this section, we will create a basic Sigstore key pair and configure
Podman to push and sign the image while storing the signature in a
staging store.[]{.sentence-end} For the sake of clarity, we will run a
registry using the basic Docker Registry V2 container image without any
customization.

Before testing the[]{#Chapter_10.xhtml#idx_547112f7 .index-entry
index-entry="container images:signing, with Podman"} image pull and
signature validation workflow, we[]{#Chapter_10.xhtml#idx_725bdc6e
.index-entry index-entry="container images:signing, with Sigstore"} will
look at how Podman handles the []{#Chapter_10.xhtml#idx_a35bd95d
.index-entry index-entry="Sigstore:container images, signing"}signature
as an **OCI artifact** or via a[]{#Chapter_10.xhtml#idx_5d0fcdf8
.index-entry index-entry="OCI artifact"}
separate[]{#Chapter_10.xhtml#idx_3aeb52c2 .index-entry
index-entry="Podman:container images, signing"} web server.

To create image signatures with **Sigstore**, we
[]{#Chapter_10.xhtml#idx_89c89166 .index-entry
index-entry="Sigstore"}need to create a valid key pair or use an
existing one.[]{.sentence-end} For this reason, we will provide a short
recap on asymmetric key pairs to help you understand how image
signatures work.

A key pair is composed of a private key and a public
key.[]{.sentence-end} The public key can be shared universally, while
[]{#Chapter_10.xhtml#idx_31a5f24a .index-entry
index-entry="private key"}the private key is kept private and never
shared with anybody.[]{.sentence-end} The **private key** is used
by[]{#Chapter_10.xhtml#idx_0e53eb66 .index-entry index-entry="sender"}
the **sender** (the image builder) to sign the image.[]{.sentence-end}
In this way, anyone with access to the []{#Chapter_10.xhtml#idx_a43234e2
.index-entry index-entry="public key"}corresponding **public key** can
verify that the image was not tampered with and truly originated from
the sender.

We can easily translate this concept into container images: the image
owner who pushes it to the remote registry can sign it using a keypair
and store the detached signature on a signature store that is publicly
accessible by users.[]{.sentence-end} Here, the signature is separated
by the image itself -- the registry will store the image blobs while the
signature store will hold and expose the image signatures.

Users who are pulling the image will be able to validate the image
signature using the previously shared public key.

Now, let\'s []{#Chapter_10.xhtml#idx_6fdb0586 .index-entry
index-entry="Cosign"}go back to creating the Sigstore key
pair.[]{.sentence-end} We are going to create a simple one with the
**Cosign** utility, a lightweight tool designed specifically for signing
and verifying container images.[]{.sentence-end} It
[]{#Chapter_10.xhtml#idx_638faa7d .index-entry
index-entry="Open Container Initiative (OCI)"}adheres to **Open
Container Initiative** (**OCI**) standards, ensuring compatibility and
seamless integration with popular container registries.

Let\'s start with Cosign installation and
configuration.[]{.sentence-end} As usual, there are several installation
methods available; here, we are reporting the most structured way to
install it via a package manager in your operating system:

``` {.programlisting .snippet-con}
 $ LATEST_VERSION=$(curl https://api.github.com/repos/sigstore/cosign/releases/latest | grep tag_name | cut -d : -f2 | tr -d "v\", ")
 $ curl -O -L "https://github.com/sigstore/cosign/releases/latest/download/cosign-${LATEST_VERSION}-1.x86_64.rpm"
 $ sudo rpm -ivh cosign-${LATEST_VERSION}-1.x86_64.rpm
```

We can []{#Chapter_10.xhtml#idx_4f3b59dd .index-entry
index-entry="Sigstore:container images, signing"}check if the
[]{#Chapter_10.xhtml#idx_9ef2ed54 .index-entry
index-entry="container images:signing, with Sigstore"}installation was
[]{#Chapter_10.xhtml#idx_7189c41f .index-entry
index-entry="Podman:container images, signing"}successful by running the
following []{#Chapter_10.xhtml#idx_aaac2101 .index-entry
index-entry="container images:signing, with Podman"}command:

``` {.programlisting .snippet-con}
 $ cosign version
...
cosign: A tool for Container Signing, Verification and Storage in an OCI registry.

GitVersion:    v3.0.4
GitCommit:     6832fba4928c1ad69400235bbc41212de5006176
GitTreeState:  clean
BuildDate:     2026-01-09T21:17:16Z
GoVersion:     go1.25.5
Compiler:      gc
Platform:      linux/amd64
```

We are now ready to generate our first key pair by running the following
command:

``` {.programlisting .snippet-con}
$  cosign generate-key-pair
```

The preceding command will ask you to provide a **passphrase** to
[]{#Chapter_10.xhtml#idx_ca8ee077 .index-entry
index-entry="passphrase"}protect your private key.[]{.sentence-end}
Unlike GPG, which manages keys in a hidden system database, Cosign
simply generates two files in your current directory.

The key pair\'s output should be similar to the following:

``` {.programlisting .snippet-con}
 $ cosign generate-key-pair
Enter password for private key:
Enter password for private key again:
Private key written to cosign.key
Public key written to cosign.pub
```

The files are already in a []{#Chapter_10.xhtml#idx_b0a7ccbe
.index-entry index-entry="Privacy Enhanced Mail (PEM)"}standard
**Privacy Enhanced Mail** (**PEM**) format, and there is no need to
perform extra export steps.[]{.sentence-end} The
`cosign.pub`{.inlineCode} file will be useful later, when we define the
image signatures\' verification policy.[]{.sentence-end} The
`cosign.key`{.inlineCode} file contains your private key and must be
kept secure, as it is used to sign your images during the push process.

Once the key pair has[]{#Chapter_10.xhtml#idx_3d473f18 .index-entry
index-entry="container images:signing, with Podman"} been generated, we
can create a basic registry []{#Chapter_10.xhtml#idx_86772534
.index-entry index-entry="container images:signing, with Sigstore"}that
will host our container images.[]{.sentence-end} To
do[]{#Chapter_10.xhtml#idx_81bf9638 .index-entry
index-entry="Sigstore:container images, signing"} so, we will reuse the
basic example[]{#Chapter_10.xhtml#idx_6f92bf1e .index-entry
index-entry="Podman:container images, signing"} from *Chapter* *9*,
*Pushing* *Images* *to* *a* *Container* *Registry*, and run the
following command as root:

``` {.programlisting .snippet-con}
$ mkdir ./registry_data
$ podman run -d \
   --name local_registry \
   -p 5000:5000 \
   -v ./registry_data:/var/lib/registry:z \
   --restart=always registry:2
```

We now have a local registry without authentication that can be used to
push the test images.[]{.sentence-end} As we mentioned previously, the
registry is unaware of the image\'s detached signature.

Podman must be able to write signatures on a staging
sigstore.[]{.sentence-end} There is already a default configuration in
the `/etc/containers/registries.d/default.yaml`{.inlineCode} file, which
looks as follows:

``` {.programlisting .snippet-code}
default-docker:
#  sigstore: file:///var/lib/containers/sigstore
  sigstore-staging: file:///var/lib/containers/sigstore
  use-sigstore-attachments: true
```

The `sigstore-staging`{.inlineCode} path is where Podman writes image
signatures; this path must be a writable folder, and it is possible to
customize it or keep the default configuration as is.[]{.sentence-end}
The `use-sigstore-attachments: true`{.inlineCode} option gives Podman
permissions to upload sigstores as sidecar files compatible with the OCI
artifact format.

For rootless users, defining a custom path in the home directory allows
Podman to successfully write signatures without requiring elevated
privileges.

If we want to create multiple user-related sigstores, we can create the
`$HOME/.config/containers/registries.d/default.yaml`{.inlineCode} file
and define a custom `sigstore-staging`{.inlineCode} path in the user\'s
home directory, following the same syntax that was shown in the previous
example.[]{.sentence-end} This will allow users to run Podman in
rootless mode and successfully write to their sigstore.

 note
**Important**

In Podman 5.x, the preferred method is to store signatures directly in
the registry as OCI artifacts.[]{.sentence-end} This removes the need to
manage a separate *sigstore* directory or worry about cross-user
filesystem permissions.


Since we want to []{#Chapter_10.xhtml#idx_91845d50 .index-entry
index-entry="container images:signing, with Podman"}demonstrate the
native integration between []{#Chapter_10.xhtml#idx_9127120c
.index-entry
index-entry="container images:signing, with Sigstore"}Podman and Cosign,
we will use the keys we[]{#Chapter_10.xhtml#idx_9642b985 .index-entry
index-entry="Sigstore:container images, signing"} generated to sign the
image during the push process.[]{.sentence-end}
Unlike[]{#Chapter_10.xhtml#idx_ce3d09a6 .index-entry
index-entry="Podman:container images, signing"} older GPG-based
workflows, Podman 5.x allows rootless users to sign and push images
seamlessly as long as they have access to their
`cosign.key`{.inlineCode} file.

The following example shows the Dockerfile of a custom
`httpd`{.inlineCode} image that\'s been built using UBI 9:

`Chapter11/image_signature/Dockerfile`{.inlineCode}

``` {.programlisting .snippet-code}
FROM registry.access.redhat.com/ubi9
# Update image and install httpd
RUN yum install -y httpd && yum clean all
# Expose the default httpd port 80
EXPOSE 80
# Run the httpd
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
```

To build the image, we can run the following command:

``` {.programlisting .snippet-con}
$ cd Chapter11/image_signature
$ podman build -t custom_httpd .
```

Now, we can tag the image with the local registry name:

``` {.programlisting .snippet-con}
$ podman tag custom_httpd localhost:5000/custom_httpd
```

Finally, it\'s time to push the []{#Chapter_10.xhtml#idx_529c9863
.index-entry index-entry="container images:signing, with Sigstore"}image
to the temporary registry and sign it []{#Chapter_10.xhtml#idx_822721de
.index-entry index-entry="container images:signing, with Podman"}using
the generated key pair.[]{.sentence-end} The `--sign-by`{.inlineCode}
option []{#Chapter_10.xhtml#idx_04073a69 .index-entry
index-entry="Podman:container images, signing"}allows users to pass a
valid key pair that\'s been identified
by[]{#Chapter_10.xhtml#idx_331de19d .index-entry
index-entry="Sigstore:container images, signing"} the user\'s email:

``` {.programlisting .snippet-con}
$ podman push \
   --tls-verify=false \
   --sign-by-sigstore-private-key ./cosign.key \
   localhost:5000/custom_httpd
Getting image source signatures
Copying blob 3ba8c926eef9 done
Copying blob a59107c02e1f done
Copying blob 352ba846236b done
Copying config 569b015109 done
Writing manifest to image destination
Creating signature: Signing image using a sigstore signature

Storing signatures
```

The preceding[]{#Chapter_10.xhtml#idx_259b9125 .index-entry
index-entry="container images:signing, with Podman"} code successfully
pushed the image blobs to the []{#Chapter_10.xhtml#idx_27b9410f
.index-entry
index-entry="container images:signing, with Sigstore"}registry and
stored the image signature.[]{.sentence-end} With that,
we[]{#Chapter_10.xhtml#idx_14c44362 .index-entry
index-entry="Sigstore:container images, signing"} have successfully
pushed and signed the []{#Chapter_10.xhtml#idx_333e682c .index-entry
index-entry="Podman:container images, signing"}image, making it more
secure for future use.[]{.sentence-end} Now, let\'s learn how to
configure Podman to retrieve signed images.

## Configuring Podman to pull signed images {#Chapter_10.xhtml#h2_256 .heading-2}

To successfully[]{#Chapter_10.xhtml#idx_bad368ea .index-entry
index-entry="Podman:configuring, to pull signed images"} pull a signed
image, Podman must be able to retrieve the signature from a signature
store and have access to a public key to verify it.

In the modern Sigstore workflow used by Podman 5.x, the registry is
*signature aware*.[]{.sentence-end} This means that when you push an
image with a signature, Podman stores that signature directly in the
registry as an OCI artifact (a sidecar object).

Because the signature is now stored alongside the image, we no longer
need to maintain an external web server or a separate directory to host
signature files.[]{.sentence-end} This greatly simplifies the DevOps
pipeline, as the registry becomes the single source of truth for both
the image layers and their cryptographic proof.

Now, let\'s configure Podman for image pulling by defining which
registries are allowed to use this feature.[]{.sentence-end} Unlike the
legacy *detached* model, Podman 5.x can retrieve signatures directly
from the registry.[]{.sentence-end} To ensure Podman looks for these
integrated signatures, we edit the
`/etc/containers/registries.d/default.yaml`{.inlineCode} file to enable
Sigstore attachments:

``` {.programlisting .snippet-code}
docker:
  localhost:5000:
    use-sigstore-attachments: true
  http://localhost:8080/
```

While we are using integrated signatures for our local registry, you can
still define external signature stores for other
registries.[]{.sentence-end} For example, the following code shows how
Podman locates signatures for the public Red Hat registry:

``` {.programlisting .snippet-code}
docker:
  registry.access.redhat.com:
    sigstore: https://access.redhat.com/webassets/docker/content/sigstore
```

Before we test the image pulls, we must implement the public key that\'s
used by Podman to verify the signatures.[]{.sentence-end} This public
key (`cosign.pub`{.inlineCode}) must be stored in the host that pulls
the image and belongs to the key pair that\'s used to sign the image.

The configuration []{#Chapter_10.xhtml#idx_10141e62 .index-entry
index-entry="Podman:configuring, to pull signed images"}file that\'s
used to enforce these security rules is
`/etc/containers/policy.json`{.inlineCode}.

The following code shows a custom configuration that requires a valid
Sigstore signature for any image pulled from
`localhost`{.inlineCode}`:5000`{.inlineCode}:

``` {.programlisting .snippet-code}
{
    "default": [
        {
            "type": "insecureAcceptAnything"
        }
    ],
    "transports": {
        "docker": {
            "localhost:5000": [
                {
                    "type": " sigstoreSigned ",
                   
                    "keyPath": "/etc/pki/containers/cosign.pub",
                    "signedIdentity": { "type": "matchRepository" }
                }
            ]
        },
        "docker-daemon": {
            "": [
                {
                    "type": "insecureAcceptAnything"
                }
            ]
        }
    }
}
```

To verify the signatures of images that have been pulled from
`localhost`{.inlineCode}`:5000`{.inlineCode}, we can use a public key
that\'s stored in the path defined by the `keyPath`{.inlineCode}
field.[]{.sentence-end} The public key must exist in the defined path
and be readable by Podman.

It is also important to note the `insecureAcceptAnything`{.inlineCode}
instruction in the `default`{.inlineCode} section.[]{.sentence-end} The
`default`{.inlineCode} key represents the global fallback policy for
Podman.[]{.sentence-end} If it tries to pull an image and doesn\'t find
a specific rule for that registry (like the one we created for
`localhost`{.inlineCode}`:5000`{.inlineCode}), it defaults to this
instruction, which tells Podman not to perform any cryptographic
checks.[]{.sentence-end} In a security context, this line is the
equivalent of leaving your front door wide open and hanging a *Welcome*
sign for everyone.

Most Linux distributions []{#Chapter_10.xhtml#idx_2ddb75fe .index-entry
index-entry="Podman:configuring, to pull signed images"}ship with this
configuration enabled by default to ensure a smooth *out-of-the-box*
experience.

Now, we are ready to test the image pull and verify its
signature.[]{.sentence-end} Ensure your `cosign.pub`{.inlineCode} file
is in the path defined in your policy, then run the following:

``` {.programlisting .snippet-con}
$ podman pull --tls-verify=false localhost:5000/custom_httpd
Getting image source signatures
Checking if image destination supports signatures
Copying blob 23fdb56daf15 skipped: already exists
Copying blob d4f13fad8263 skipped: already exists
Copying blob 96b0fdd0552f done
Copying config 569b015109 done
Writing manifest to image destination
Storing signatures
569b015109d457ae5fabb969fd0dc3cce10a3e6683ab60dc10505fc2d68e769f
```

The image was []{#Chapter_10.xhtml#idx_a3a4e5c6 .index-entry
index-entry="Podman:configuring, to pull signed images"}successfully
pulled into the local store after signature verification using the
public key provided.[]{.sentence-end} If the signature had been missing
or tampered with, Podman would have rejected the pull immediately.

Now, let\'s see how Podman behaves when it is unable to correctly verify
the signature.

## Testing signature verification failures {#Chapter_10.xhtml#h2_257 .heading-2}

A security policy is only as []{#Chapter_10.xhtml#idx_31b84d20
.index-entry index-entry="signature verification failures:testing"}good
as its ability to block untrusted content.[]{.sentence-end} To verify
that our policy is working, we should test what happens when
verification fails.[]{.sentence-end} First, let\'s remove our local
image to ensure Podman is forced to pull it from the registry and
perform the check:

``` {.programlisting .snippet-con}
$ podman rmi localhost:5000/custom_httpd
```

What happens if an image exists in the registry but has not been signed?
We can simulate this by pushing the same image with a different tag
without using the `--sign-by`{.inlineCode} flag and then trying to pull
the image again:

``` {.programlisting .snippet-con}
$ podman tag custom_httpd localhost:5000/custom_httpd:unsigned
$ podman push --tls-verify=false localhost:5000/custom_httpd:unsigned
$ podman pull --tls-verify=false localhost:5000/custom_httpd:unsigned
Error: unable to copy from source docker://localhost:5000/custom_httpd:unsigned: Source image rejected: A signature was required, but no signature exists
```

The preceding error demonstrates that since our
`policy.json`{.inlineCode} requires a signature for any image from
`localhost`{.inlineCode}`:5000`{.inlineCode}, Podman will automatically
reject it.

Another common failure occurs when the public key used for verification
does not match the private key used for signing.[]{.sentence-end} To
test this, we can modify our
`/etc/containers/p`{.inlineCode}`olicy.json`{.inlineCode} file to point
to an incorrect public key.[]{.sentence-end} For example, we could
temporarily point `keyPath`{.inlineCode} to a public key from another
project or generate a dummy key:

``` {.programlisting .snippet-con}
# Generate a dummy "wrong" key
$ cosign generate-key-pair --output-key-prefix dummy
```

Update `/etc/containers/policy.json`{.inlineCode} to use the
`dummy.pub`{.inlineCode} public key, then try to pull the original
signed image:

``` {.programlisting .snippet-con}
$ podman pull --tls-verify=false localhost:5000/custom_httpd
Error: Source image rejected: Invalid signature: key mismatch
```

These errors confirm that Podman is actively enforcing the trust
relationship.[]{.sentence-end} In a production environment, this
prevents *man-in-the-middle* attacks or the accidental deployment of
unvetted images.

 note
**Important**

Do not forget to restore the valid public key in the
`/etc/containers/policy.json`{.inlineCode} file before proceeding with
the following examples.


Podman offers[]{#Chapter_10.xhtml#idx_d651880e .index-entry
index-entry="signature verification failures:testing"} even more
granular control over these policies and also offers dedicated CLI
commands to help you customize security policies, which we\'ll see in
the next subsection.

## Managing keys with Podman image trust commands {#Chapter_10.xhtml#h2_258 .heading-2}

It is possible to edit []{#Chapter_10.xhtml#idx_0a39816b .index-entry
index-entry="Podman image trust commands:keys, managing"}the
`/etc/containers/policy.json`{.inlineCode} file and modify its JSON
objects to add or remove configurations for
[]{#Chapter_10.xhtml#idx_4f60b969 .index-entry
index-entry="keys:managing, with Podman image trust commands"}dedicated
registries.[]{.sentence-end} However, manual editing can be error-prone
and hard to automate in a fast-moving DevOps environment.

Alternatively, we can use the
`podman`{.inlineCode}` `{.inlineCode}`image`{.inlineCode}` `{.inlineCode}`trust`{.inlineCode}
command to dump or modify the current configuration.

The following code shows how to print the current configuration with the
`podman`{.inlineCode}` `{.inlineCode}`image`{.inlineCode}` `{.inlineCode}`trust`{.inlineCode}` `{.inlineCode}`show`{.inlineCode}
command:

``` {.programlisting .snippet-con}
$ podman image trust show
http://localhost:8080/
TRANSPORT      NAME            TYPE            ID          STORE
all            default         accept                     
repository     localhost:5000  sigstoreSigned  N/A        
docker-daemon                  accept                     
```

It is also possible to configure new trusts.[]{.sentence-end} For
example, we can add the Red Hat public GPG key to check the signature of
UBI images.

First, we need to download the Red Hat public key:

``` {.programlisting .snippet-con}
$ sudo wget -O /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat \
  https://www.redhat.com/security/data/fd431d51.txt
```

 note
**Note**

Red Hat\'s product signing keys, including the one that was used in this
example, can be found at
[[https://access.redhat.com/security/team/key]{.url}](https://access.redhat.com/security/team/key){style="text-decoration: none;"}.


After[]{#Chapter_10.xhtml#idx_7f19a49a .index-entry
index-entry="Podman image trust commands:keys, managing"} downloading
the key, we must configure []{#Chapter_10.xhtml#idx_dbd1f214
.index-entry
index-entry="keys:managing, with Podman image trust commands"}the image
trust for UBI 9 images that have been pulled from
`registry.access.redhat.com`{.inlineCode} using the
`podman`{.inlineCode}` `{.inlineCode}`image`{.inlineCode}` `{.inlineCode}`trust`{.inlineCode}` `{.inlineCode}`set`{.inlineCode}
command:

``` {.programlisting .snippet-con}
$ sudo podman image trust set -f /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat registry.access.redhat.com/ubi9
```

After running the preceding command, the
`/etc/containers/policy.json`{.inlineCode} file will change, as follows:

``` {.programlisting .snippet-code}
{
    "default": [
        {
            "type": "insecureAcceptAnything"
        }
    ],
    "transports": {
        "docker": {
            "localhost:5000": [
                {
                   
                     "type": "sigstoreSigned",
                    "keyPath": "/etc/pki/containers/cosign.pub",
                    "signedIdentity": {
                        "type": "matchRepository"
                    }
                }
            ],
            "registry.access.redhat.com/ubi9": [
                {
                    "type": "signedBy",
                    "keyType": "GPGKeys",
                    "keyPath": "/etc/pki/rpm-gpg/RPM-GPG-KEY-redhat"
                }
            ]
        },
        "docker-daemon": {
            "": [
                {
                    "type": "insecureAcceptAnything"
                }
            ]
        }
    }
```

Note that the entry that\'s related to
`registry.access.redhat.com/`{.inlineCode}`ubi`{.inlineCode}`9`{.inlineCode}
and the public key that was used []{#Chapter_10.xhtml#idx_802088c9
.index-entry index-entry="Podman image trust commands:keys, managing"}to
verify the image signatures have been []{#Chapter_10.xhtml#idx_fd87eed6
.index-entry
index-entry="keys:managing, with Podman image trust commands"}added to
the file.

The Red Hat sigstore configuration for this registry is already
available in the
`/etc/containers/registries.d/`{.inlineCode}`registry.access.redhat.com`{.inlineCode}`.`{.inlineCode}`yaml`{.inlineCode}
file, installed with the Podman package:

``` {.programlisting .snippet-code}
docker:
  registry.access.redhat.com:
    sigstore: https://access.redhat.com/webassets/docker/content/sigstore
```

 note
**Tip**

Red Hat related registry files have been included in the
`github.com/`{.inlineCode}`containers/common`{.inlineCode} repository to
allow easier integration with Red Hat\'s ecosystem.

It is possible to create custom registry configuration files for
different providers in the `/etc/containers/registries.d`{.inlineCode}
folder.[]{.sentence-end} For example, the preceding example could be
defined in a dedicated
`/etc/containers/registries.d/redhat.yaml`{.inlineCode}
file.[]{.sentence-end} This allows you to easily maintain and version
registry sigstore configurations.


From now on, every []{#Chapter_10.xhtml#idx_1f1f35f8 .index-entry
index-entry="Podman image trust commands:keys, managing"}time a UBI9
image is pulled []{#Chapter_10.xhtml#idx_0e70b724 .index-entry
index-entry="keys:managing, with Podman image trust commands"}from
`registry.access.redhat.com`{.inlineCode}, its signature will be pulled
from the Red Hat signature store and validated using the provided public
key.

So far, we have looked at examples of managing keys in Podman, but it is
also possible to manage signature verification with
Skopeo.[]{.sentence-end} In the next subsection, we are going to look at
some basic examples.

## Managing signatures with Skopeo {#Chapter_10.xhtml#h2_259 .heading-2}

We can verify an image signature []{#Chapter_10.xhtml#idx_faa5c07e
.index-entry index-entry="Skopeo"}using **Skopeo** when we\'re pulling
an image from a valid transport.

The following []{#Chapter_10.xhtml#idx_f548bbed .index-entry
index-entry="Skopeo:signatures, managing"}example uses the
`skopeo`{.inlineCode}` `{.inlineCode}`copy`{.inlineCode} command to pull
the image from []{#Chapter_10.xhtml#idx_c73c1d49 .index-entry
index-entry="signatures:managing, with Skopeo"}our registry to the local
store.[]{.sentence-end} This command has the same effects as using a
`podman`{.inlineCode}` `{.inlineCode}`pull`{.inlineCode} command but
allows more control over the source and destination transports:

``` {.programlisting .snippet-con}
$ skopeo copy --src-tls-verify=false \
  docker://localhost:5000/custom_httpd \
  containers-storage:localhost:5000/custom_httpd
```

Skopeo []{#Chapter_10.xhtml#idx_0f0c2746 .index-entry
index-entry="Skopeo:signatures, managing"}does not need any further
configuration because it automatically []{#Chapter_10.xhtml#idx_79d99b4e
.index-entry index-entry="signatures:managing, with Skopeo"}references
the public key path defined in
`/etc/containers/policy.json`{.inlineCode}.[]{.sentence-end} If the
signature in the registry does not match our public key, Skopeo will
refuse to copy the image.

We can also use Skopeo to sign an image with our private key before
copying it to a transport:

``` {.programlisting .snippet-con}
$ skopeo copy \
   --dest-tls-verify=false \
   --sign-by-sigstore-private-key ./cosign.key \
   containers-storage:localhost:5000/custom_httpd \
   docker://localhost:5000/custom_httpd
```

In this section, we learned how to verify image signatures and avoid
potential MITM attacks.[]{.sentence-end} In the next section, we will
see how to simplify the keys\' creation process with the help of Skopeo.

## Generating a Sigstore key with Skopeo {#Chapter_10.xhtml#h2_260 .heading-2}

Skopeo provides []{#Chapter_10.xhtml#idx_b35d65cb .index-entry
index-entry="Sigstore key:generating, with Skopeo"}a useful option that
simplifies the creation of keys for[]{#Chapter_10.xhtml#idx_2e721061
.index-entry index-entry="Skopeo:Sigstore key, generating"} signing
container images.

First of all, let\'s create a directory for holding the public and
private keys we are going to generate:

``` {.programlisting .snippet-con}
$ mkdir .skopeo
```

We are now ready to leverage Skopeo for the keys\' creation process:

``` {.programlisting .snippet-con}
$ skopeo generate-sigstore-key --output-prefix .skopeo/sig-ubi8-httpd
Passphrase for key .skopeo/sig-ubi8-httpd.private:
Key written to ".skopeo/sig-ubi8-httpd.private" and ".skopeo/sig-ubi8-httpd.pub"
```

The []{#Chapter_10.xhtml#idx_033dcb09 .index-entry
index-entry="Sigstore key:generating, with Skopeo"}command will also ask
for a passphrase in case we want to []{#Chapter_10.xhtml#idx_b88dc48c
.index-entry index-entry="Skopeo:Sigstore key, generating"}increase the
security level of the key.[]{.sentence-end} Once we generate a brand-new
couple of private and public keys, we can then follow the previous
sections for signing the container images and trust the content pulled
by container registries.

We just learned the basics of signing and managing container
images.[]{.sentence-end} In the next section, we will take a look at how
the open source community tried to help in this process by creating a
set of tools for image signature management.

## Using Rekor and Cosign to manage image signatures {#Chapter_10.xhtml#h2_261 .heading-2}

This section explores how to leverage Cosign and Rekor to establish a
robust and trustworthy system for managing container image signatures,
significantly bolstering the security of your software supply chain.

Before diving into the implementation, let\'s clarify the roles of these
two key players:

-   **Cosign**: A[]{#Chapter_10.xhtml#idx_2b0cff55 .index-entry
    index-entry="Cosign"} lightweight tool designed specifically for
    signing and verifying container images.[]{.sentence-end} It adheres
    []{#Chapter_10.xhtml#idx_aac688a9 .index-entry
    index-entry="Open Container Initiative (OCI)"}to **Open**
    **Container** **Initiative** (**OCI**) standards, ensuring
    compatibility and seamless integration with popular container
    registries.
-   **Rekor**: A []{#Chapter_10.xhtml#idx_c959fe80 .index-entry
    index-entry="Rekor"}transparency log server that provides an
    immutable record of all metadata generated during the signing
    process.[]{.sentence-end} This immutability guarantees the integrity
    of signatures, preventing any retroactive tampering or revocation.

We already []{#Chapter_10.xhtml#idx_8f55a338 .index-entry
index-entry="Rekor:image signatures, managing"}have our
`cosign`{.inlineCode} binary installed on the system.[]{.sentence-end}
For this[]{#Chapter_10.xhtml#idx_9486cc61 .index-entry
index-entry="Cosign:image signatures, managing"} example, we are not
going to deploy a Rekor server, even[]{#Chapter_10.xhtml#idx_61c98368
.index-entry index-entry="image signatures:managing, with Rekor"} though
the project page explains in detail all
the[]{#Chapter_10.xhtml#idx_14fce412 .index-entry
index-entry="image signatures:managing, with Cosign"} prerequisites
needed.[]{.sentence-end} Instead, we are going to leverage the public
Rekor server offered by the open source community, available at
[[https://rekor.sigstore.dev]{.url}](https://rekor.sigstore.dev){style="text-decoration: none;"}.

We are now ready to test both Cosign and Rekor for signing our first
container image with the help of these tools.

We are going to sign the container image we created earlier,
`custom_httpd`{.inlineCode}, through the `sig-ubi8-httpd`{.inlineCode}
key created through Skopeo.[]{.sentence-end} We execute the signature
process through the following command:

``` {.programlisting .snippet-con}
 $ cosign sign --key .cosign.key quay.io/alezzandro/ubi9-httpd@sha256:cda838d6ef3ddf951c1c7f1086d1aa1ed895159a4e794b8629da66b35b94c83f
Enter password for private key:
...
tlog entry created with index: 165166231
Pushing signature to: quay.io/alezzandro/ubi9-httpd
```

As shown in the previous command, the best practice is to sign a
container image based on its digest.[]{.sentence-end} This avoids any
mismatch if we override an image tag with a newer release.

 note
**Note**

The previous command requires that the container image resides in a
container registry that is also capable of handling the image
signatures.[]{.sentence-end} You should be authenticated against the
container image registry, and you should have the right permissions for
executing these actions.


Behind the[]{#Chapter_10.xhtml#idx_d8f65457 .index-entry
index-entry="Rekor:image signatures, managing"} scenes, Cosign generates
a cryptographic signature []{#Chapter_10.xhtml#idx_6d884a24 .index-entry
index-entry="Cosign:image signatures, managing"}for your container image
and creates an entry in[]{#Chapter_10.xhtml#idx_b5c4587c .index-entry
index-entry="image signatures:managing, with Rekor"} the Rekor public
server log, recording the[]{#Chapter_10.xhtml#idx_802ea7c9 .index-entry
index-entry="image signatures:managing, with Cosign"} signature along
with crucial metadata such as the image digest, public key used for
signing, and timestamp.

Let\'s inspect the image signature and its log stored on the Rekor
public server.

First, we are going to download one of the latest Rekor command-line
interfaces from the public GitHub repository:

``` {.programlisting .snippet-con}
 $ curl -O -L https://github.com/sigstore/rekor/releases/download/v1.3.8/rekor-cli-linux-amd64
 $ chmod +x rekor-cli-linux-amd64
```

We are now ready to query the Rekor public server with the following
command:

``` {.programlisting .snippet-con}
 $ ./rekor-cli-linux-amd64 get --log-index 165166231
LogID: c0d23d6ad406973f9559f3ba2d1ca01f84147d8ffc5b8445c224f98b9591801d
Index: 165166231
IntegratedTime: 2025-01-24T11:20:22Z
UUID: 108e9186e8c5677af3fa03b736ca86f4605e0229fb620863ff8d0b550db9e43f0aad3b2da8c0df4f
Body: {
... omitted output ...
      }
    }
  }
}
```

Finally, we[]{#Chapter_10.xhtml#idx_351fe156 .index-entry
index-entry="Rekor:image signatures, managing"} can verify the
image[]{#Chapter_10.xhtml#idx_5a768240 .index-entry
index-entry="image signatures:managing, with Rekor"} signature with
[]{#Chapter_10.xhtml#idx_29f99a5a .index-entry
index-entry="image signatures:managing, with Cosign"}the Cosign
command-line []{#Chapter_10.xhtml#idx_487ae5bd .index-entry
index-entry="Cosign:image signatures, managing"}tool:

``` {.programlisting .snippet-con}
 $ cosign verify --key ./cosign.pub quay.io/alezzandro/ubi9-httpd@sha256:cda838d6ef3ddf951c1c7f1086d1aa1ed895159a4e794b8629da66b35b94c83f

Verification for quay.io/alezzandro/ubi9-httpd@sha256:cda838d6ef3ddf951c1c7f1086d1aa1ed895159a4e794b8629da66b35b94c83f --
The following checks were performed on each of these signatures:
  - The cosign claims were validated
  - Existence of the claims in the transparency log was verified offline
  - The signatures were verified against the specified public key
    
[{"critical":{"identity":{"docker-reference":"quay.io/alezzandro/ubi9-httpd"},"image":{"docker-manifest-digest":"sha256:cda838d6ef3ddf951c1c7f1086d1aa1ed895159a4e794b8629da66b35b94c83f"},"type":"cosign container image signature"},
... omitted output
```

As shown before, the Rekor log provides an undeniable audit trail,
preventing anyone from denying their involvement in signing an
image.[]{.sentence-end} Anyone can independently audit the Rekor log to
verify the authenticity and integrity of image
signatures.[]{.sentence-end} Rekor adds a critical layer of security,
making it significantly harder for attackers to compromise your image
signing process.

To maximize efficiency and security, we can integrate Cosign and Rekor
into our CI/CD pipelines.[]{.sentence-end} This automation ensures that
all images are signed and verified as part of your development workflow.

Finally, by []{#Chapter_10.xhtml#idx_551498c0 .index-entry
index-entry="Rekor:image signatures, managing"}combining Cosign and
Rekor, you can establish a robust []{#Chapter_10.xhtml#idx_665151c1
.index-entry index-entry="Cosign:image signatures, managing"}and
transparent system for managing
container[]{#Chapter_10.xhtml#idx_64f53a2b .index-entry
index-entry="image signatures:managing, with Rekor"} image signatures,
significantly strengthening[]{#Chapter_10.xhtml#idx_86df5871
.index-entry index-entry="image signatures:managing, with Cosign"} your
software supply chain against potential threats.

In the next section, we\'ll shift focus and learn how to execute the
container\'s runtime by customizing Linux kernel capabilities.

## Customizing Linux kernel capabilities {#Chapter_10.xhtml#h2_262 .heading-2}

Capabilities are features[]{#Chapter_10.xhtml#idx_a8162b48 .index-entry
index-entry="Linux kernel:capabilities, customizing"} that were
introduced in Linux kernel 2.2 with the purpose of splitting elevated
privileges into single units that can be arbitrarily assigned to a
process or thread.

Instead of running a process as a fully privileged instance with
effective UID 0, we can assign a limited subset of specific capabilities
to an unprivileged process.[]{.sentence-end} By providing more granular
control over the security []{#Chapter_10.xhtml#idx_f2565091 .index-entry
index-entry="Linux kernel:capabilities, customizing"}context of the
process\'s execution, this approach helps mitigate potential attack
tactics.

Before we discuss the capabilities of containers, let\'s recap how they
work in a Linux system so that we understand their inner logic.

## Understanding Linux Kernel capabilities {#Chapter_10.xhtml#h2_263 .heading-2}

Capabilities are []{#Chapter_10.xhtml#idx_c52fde46 .index-entry
index-entry="Linux kernel:capabilities"}associated with the file
executables using extended attributes (see
`man`{.inlineCode}` `{.inlineCode}`xattr`{.inlineCode}) and are granted
to the resulting process upon an `execve()`{.inlineCode} system call,
subject to the kernel\'s capability transition rules.

The list of available capabilities is quite large and still growing; it
includes very specific actions that can be performed by a
thread.[]{.sentence-end} Some basic examples are as follows:

-   `CAP_CHOWN`{.inlineCode}: This capability allows a thread to modify
    a file\'s UID and GID.
-   `CAP_KILL`{.inlineCode}: This capability allows you to bypass the
    permission checks to send a signal to a process.
-   `CAP_MKNOD`{.inlineCode}: This capability allows you to create a
    special file with the `mknod(`{.inlineCode}`)`{.inlineCode} syscall.
-   `CAP_NET_ADMIN`{.inlineCode}: This capability allows you to operate
    various privileged actions on the system\'s network configuration,
    including changing the interface configuration, enabling/disabling
    promiscuous mode for an interface, editing routing tables, and
    enabling/disabling multicasting.
-   `CAP_NET_RAW`{.inlineCode}: This capability allows a thread to use
    RAW and PACKET sockets.[]{.sentence-end} It can be used by programs
    such as `ping`{.inlineCode} to send ICMP packets without the need
    for elevated privileges.
-   `CAP_SYS_CHROOT`{.inlineCode}: This capability allows you to use the
    `chroot(`{.inlineCode}`)`{.inlineCode} syscall and change mount
    namespaces with the `setns()`{.inlineCode} syscall.
-   `CAP_SYS_ADMIN`{.inlineCode}: This capability allows near-total root
    privileges, allowing processes to mount filesystems and bypass
    critical container isolation.[]{.sentence-end} Granting this
    capability allows containers to bypass vital isolation layers,
    effectively giving a process full control over the host\'s
    underlying kernel.
-   `CAP_DAC_OVERRIDE`{.inlineCode}: This capability allows you to
    bypass **discretionary** **access** **control** (**DAC**)
    checks[]{#Chapter_10.xhtml#idx_951ad619 .index-entry
    index-entry="discretionary access control (DAC)"} for file read,
    write, and execution.
-   `CAP_BPF`{.inlineCode}: Introduced in kernel 5.8, this capability
    employs privileged BPF operations.

For more details and an extensive []{#Chapter_10.xhtml#idx_71306e73
.index-entry index-entry="Linux kernel:capabilities"}list of available
capabilities, see the relevant man page
(`man`{.inlineCode}` `{.inlineCode}`capabilities`{.inlineCode}).

To assign a capability to an executable, we can use the
`setcap`{.inlineCode} command, as shown in the following example, where
`CAP_NET_ADMIN`{.inlineCode} and `CAP_NET_RAW`{.inlineCode} are being
permitted in the `/usr/bin/ping`{.inlineCode} executable:

``` {.programlisting .snippet-con}
$ sudo setcap 'cap_net_admin,cap_net_raw+p' /usr/bin/ping
```

The `+p`{.inlineCode} flag in the preceding command indicates that the
capabilities have been set to **Permitted**.

To inspect the capabilities of a file, we can use the
`getcap`{.inlineCode} command:

``` {.programlisting .snippet-con}
$ getcap /usr/bin/ping
/usr/bin/ping cap_net_admin,cap_net_raw=p
```

See `man`{.inlineCode}` `{.inlineCode}`getcap`{.inlineCode} and
`man`{.inlineCode}` `{.inlineCode}`setcap`{.inlineCode} for more details
about these utilities.

We can inspect the active capabilities of a running process by looking
at the
`/proc/`{.inlineCode}`<`{.inlineCode}`PID`{.inlineCode}`>`{.inlineCode}`/status`{.inlineCode}
file.[]{.sentence-end} In the following code, we are launching a
`ping`{.inlineCode} command after setting the
`CAP_NET_ADMIN`{.inlineCode} and `CAP_NET_RAW`{.inlineCode}
capabilities.[]{.sentence-end} We want to launch the process in the
background and check its current capabilities:

``` {.programlisting .snippet-con}
$ ping example.com > /dev/null 2>&1 &
$ grep 'Cap.*' /proc/$(pgrep ping)/status
CapInh: 0000000000000000
CapPrm: 0000000000003000
CapEff: 0000000000000000
CapBnd: 000000ffffffffff
CapAmb: 0000000000000000
```

Here, we are[]{#Chapter_10.xhtml#idx_7c1a0338 .index-entry
index-entry="Linux kernel:capabilities"} interested in evaluating the
bitmap in the `CapPrm`{.inlineCode} field, which represents the
permitted capabilities.[]{.sentence-end} To get a user-friendly value,
we can use the `capsh`{.inlineCode} command to decode the bitmap hex
value:

``` {.programlisting .snippet-con}
$ capsh --decode=0000000000003000
0x0000000000003000=cap_net_admin,cap_net_raw
```

The result is similar to the output of the `getcap`{.inlineCode} command
in the `/usr/bin/ping`{.inlineCode} file, demonstrating that executing
the command propagated the file\'s permitted capabilities to its process
instance.

For a full list of the constants that were used to set the bitmaps, as
well as their capabilities, see the following kernel header file:
[[https://github.com/torvalds/linux/blob/master/include/uapi/linux/capability.h]{.url}](https://github.com/torvalds/linux/blob/master/include/uapi/linux/capability.h){style="text-decoration: none;"}.

 packt_tip
**Tip**

Older versions in distributions such as RHEL and CentOS used the
preceding configuration to allow `ping`{.inlineCode} to send ICMP
packets with access from all users without them being executed as
privileged processes with
`setuid`{.inlineCode}` `{.inlineCode}`0`{.inlineCode}.[]{.sentence-end}
This was an insecure approach where an attacker could leverage a
vulnerability or bug in the executable to escalate privileges and gain
control of the system.

Fedora introduced a new and more secure approach since version 31
that\'s based on using the `net.ipv4.ping_group_range`{.inlineCode}
Linux kernel parameter.[]{.sentence-end} By setting an extensive range
that covers all system groups, this parameter allows users to send ICMP
packets without the need to enable the `CAP_NET_ADMIN`{.inlineCode} and
`CAP_NET_RAW`{.inlineCode} capabilities.[]{.sentence-end} This approach
was inherited by RHEL and many other similar distributions and the new,
more secure standard.

For more details, see the following wiki page from the Fedora Project:
[[https://fedoraproject.org/wiki/Changes/EnableSysctlPingGroupRange]{.url}](https://fedoraproject.org/wiki/Changes/EnableSysctlPingGroupRange){style="text-decoration: none;"}.


Now that we\'ve provided a high-level description of the Linux kernel\'s
capabilities, let\'s learn how they are applied to containers.

## Using capabilities in containers {#Chapter_10.xhtml#h2_264 .heading-2}

Capabilities []{#Chapter_10.xhtml#idx_57eb1492 .index-entry
index-entry="Linux kernel:capabilities, using in containers"}can be
applied inside containers to allow targeted actions to take
place.[]{.sentence-end} By default, Podman runs containers using a set
of Linux kernel capabilities[]{#Chapter_10.xhtml#idx_9b2f76c3
.index-entry index-entry="containers:Linux kernel capabilities, using"}
that are defined in the
`/usr/share/containers/containers.conf`{.inlineCode}
file.[]{.sentence-end} At the time of writing, the following
capabilities are enabled and declared inside this file (with comments):

``` {.programlisting .snippet-code}
#default_capabilities = [
#    "CHOWN",
#    "DAC_OVERRIDE",
#    "FOWNER",
#    "FSETID",
#    "KILL",
#    "NET_BIND_SERVICE",
#    "SETFCAP",
#    "SETGID",
#    "SETPCAP",
#    "SETUID",
#    "SYS_CHROOT"
#]
```

We can run a simple test to verify that those capabilities have been
effectively applied to a process running inside a
container.[]{.sentence-end} For this test, we will use the official
nginx image:

``` {.programlisting .snippet-con}
$ podman run -d --name cap_test docker.io/library/nginx
$ podman exec -it cap_test sh -c 'grep Cap /proc/1/status'
CapInh: 0000000000000000    
CapPrm: 00000000800405fb
CapEff: 00000000800405fb
CapBnd: 00000000800405fb
CapAmb: 0000000000000000
```

Here is []{#Chapter_10.xhtml#idx_c31942e7 .index-entry
index-entry="Linux kernel:capabilities, using in containers"}what each
line represents for the process:

-   `CapInh`{.inlineCode} (**inheritable**): These are
    []{#Chapter_10.xhtml#idx_0da7f7e0 .index-entry
    index-entry="containers:Linux kernel capabilities, using"}capabilities
    that can be preserved across an
    `execve`{.inlineCode}.[]{.sentence-end} In this case, it\'s all
    zeros, meaning no capabilities will be automatically inherited by a
    child process unless specifically configured.
-   `CapPrm`{.inlineCode} (**permitted**): This is the *limiting
    set*.[]{.sentence-end} It defines the maximum capabilities the
    process can actually use.[]{.sentence-end} The process can move
    capabilities from this set into the *effective* set.
-   `CapEff`{.inlineCode} (**effective**): This is the set the kernel
    actually checks.[]{.sentence-end} For example, if a process tries to
    change the system clock, the kernel looks at the bit for
    `CAP_SYS_TIME`{.inlineCode} in this specific set.[]{.sentence-end}
    Since this matches `CapPrm`{.inlineCode}, the process is currently
    *active* with all its allowed privileges.
-   `CapBnd`{.inlineCode} (**bounding**): This is a mechanism used to
    limit the capabilities a process can ever gain.[]{.sentence-end} You
    cannot *promote* a capability into your *permitted* set if it isn\'t
    already in your *bounding* set.
-   `CapAmb`{.inlineCode} (**ambient**): A newer set (added in Linux
    4.3) that solves the problem of localizing capabilities for non-root
    users.[]{.sentence-end} It allows capabilities to be preserved when
    running a program that is not `setuid`{.inlineCode}.

Here, we have extracted the current capabilities from the parent nginx
process (running with `PID`{.inlineCode}` `{.inlineCode}`1`{.inlineCode}
inside the container).[]{.sentence-end} Now, we can check the bitmap
with the `capsh`{.inlineCode} utility:

``` {.programlisting .snippet-con}
$ capsh --decode=00000000800405fb
0x00000000800405fb=cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_sys_chroot,cap_setfcap
```

The preceding[]{#Chapter_10.xhtml#idx_0b27a2c9 .index-entry
index-entry="Linux kernel:capabilities, using in containers"} list of
capabilities is the same as the list []{#Chapter_10.xhtml#idx_6f4d1898
.index-entry
index-entry="containers:Linux kernel capabilities, using"}that was
defined in the default Podman configuration.[]{.sentence-end} Note that
the capabilities are applied in both rootless and rootful mode.

 note
**Note**

If you\'re curious, the capabilities for the containerized process(es)
are set up by the container runtime, which is either `runc`{.inlineCode}
or `crun`{.inlineCode}, based on the distribution.


Now that we know how capabilities are configured and applied inside
containers, let\'s learn how to customize a container\'s capabilities.

## Customizing a container\'s capabilities {#Chapter_10.xhtml#h2_265 .heading-2}

We can add or drop []{#Chapter_10.xhtml#idx_2f2096a3 .index-entry
index-entry="containers:capabilities, customizing"}capabilities either
at runtime or statically.

To statically change the default capabilities, we can simply edit the
`default_capabilities`{.inlineCode} field in the
`/usr/share/containers/containers.conf`{.inlineCode} file and add or
remove them according to our desired results.

To modify capabilities at runtime, we can use the
`-`{.inlineCode}`cap-add`{.inlineCode} and
`-`{.inlineCode}`cap-drop`{.inlineCode} options, both of which are
provided by the `podman`{.inlineCode}` `{.inlineCode}`run`{.inlineCode}
command.

The following code removes the `CAP_DAC_OVERRIDE`{.inlineCode}
capability from a container:

``` {.programlisting .snippet-con}
$ podman run -d --name cap_test2 --cap-drop=DAC_OVERRIDE docker.io/library/nginx
```

If we look at the capability bitmaps again, we will see that they were
updated accordingly:

``` {.programlisting .snippet-con}
$ podman exec cap_test2 sh -c 'grep Cap /proc/1/status'
CapInh: 0000000000000000
CapPrm: 00000000800405f9
CapEff: 00000000800405f9
CapBnd: 00000000800405f9
CapAmb: 0000000000000000
$ capsh --decode=00000000800405f9
0x00000000800405f9=cap_chown,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_sys_chroot,cap_setfcap
```

It is possible to pass the `--cap-add`{.inlineCode} and
`--cap-drop`{.inlineCode} options multiple times:

``` {.programlisting .snippet-con}
$ podman run -d --name cap_test3 \
   --cap-drop=KILL \
   --cap-drop=DAC_OVERRIDE \
   --cap-add=NET_RAW \
   --cap-add=NET_ADMIN \
   docker.io/library/nginx
```

When we\'re dealing []{#Chapter_10.xhtml#idx_cd719e41 .index-entry
index-entry="containers:capabilities, customizing"}with capabilities, we
must be careful when dropping a default capability.[]{.sentence-end} The
following code shows an error in the nginx container when dropping the
`CAP_CHOWN`{.inlineCode} capability:

``` {.programlisting .snippet-con}
$ podman run --name cap_test4 \
  --cap-drop=CHOWN \
  docker.io/library/nginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2022/01/06 23:19:39 [emerg] 1#1: chown("/var/cache/nginx/client_temp", 101) failed (1: Operation not permitted)
nginx: [emerg] chown("/var/cache/nginx/client_temp", 101) failed (1: Operation not permitted)
```

Here, the container fails.[]{.sentence-end} From the output, we can see
that the nginx process was unable to show the
`/var/cache/nginx/client_temp`{.inlineCode} directory.[]{.sentence-end}
This is a direct consequence of the `CAP_CHOWN`{.inlineCode} capability
being removed.

Capabilities can be applied to a rootless container, but they are
namespaced capabilities, which are more limited than regular ones and
are unable to perform actions that would alter the host
system.[]{.sentence-end} For []{#Chapter_10.xhtml#idx_281c4246
.index-entry index-entry="containers:capabilities, customizing"}example,
if we try to apply the `CAP_MKNOD`{.inlineCode} capability to a rootless
container, any attempt to create a special file inside a rootless
container won\'t be allowed by the kernel:

``` {.programlisting .snippet-con}
$ podman run -it --cap-add=MKNOD \
  docker.io/library/busybox /bin/sh
/ # mkdir -p /test/dev
/ # mknod -m 666 /test/dev/urandom c 1 8
mknod: /test/dev/urandom: Operation not permitted
```

Instead, if we run the container with elevated root privileges, the
capability can be assigned successfully:

``` {.programlisting .snippet-con}
# podman run -it --cap-add=MKNOD \
  docker.io/library/busybox /bin/sh
/ # mkdir -p /test/dev
/ # mknod -m 666 /test/dev/urandom c 1 8
/ # stat /test/dev/urandom
File: /test/dev/urandom
  Size: 0          Blocks: 0          IO Block: 4096   character special file
Device: 31h/49d Inode: 530019      Links: 1     Device type: 1,8
Access: (0666/crw-rw-rw-)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2022-01-06 23:50:06.056650747 +0000
Modify: 2022-01-06 23:50:06.056650747 +0000
Change: 2022-01-06 23:50:06.056650747 +0000
```

 note
**Note**

Generally, adding capabilities to containers implies enlarging the
potential attack surface that a malicious attacker could
use.[]{.sentence-end} If it\'s not necessary, it is a good practice to
keep the default capabilities and drop the unwanted ones once the
potential side effects have been analyzed.


In this section, we learned how to manage capabilities inside
containers.[]{.sentence-end} However, capabilities are not the only
security aspect to consider when you\'re securing
containers.[]{.sentence-end} SELinux, as we will learn in the next
section, has a crucial role in guaranteeing container isolation.

# Understanding SELinux interaction with containers {#Chapter_10.xhtml#h1_266 .heading-1}

In this section, we will discuss SELinux
[]{#Chapter_10.xhtml#idx_60b2409d .index-entry
index-entry="Udica"}policies and introduce **Udica**, a tool that\'s
used to generate SELinux profiles for containers.

SELinux works directly in kernel space and manages object isolation
while following a least-privilege model that contains a series of
**policies** that []{#Chapter_10.xhtml#idx_0b3cc4fb .index-entry
index-entry="policies"}can handle enforcing or
exceptions.[]{.sentence-end} To define these objects, SELinux uses
labels that[]{#Chapter_10.xhtml#idx_0e2a5287 .index-entry
index-entry="types"} define **types**.[]{.sentence-end} By default,
SELinux works in **Enforcing** mode,
denying[]{#Chapter_10.xhtml#idx_cc33ad43 .index-entry
index-entry="Enforcing mode"} access to resources with a series of
exceptions defined by policies.[]{.sentence-end} To disable
**Enforcing** mode, SELinux can be put in **Permissive** mode, where
[]{#Chapter_10.xhtml#idx_95c0d167 .index-entry
index-entry="Permissive mode"}violations are only audited, without them
being blocked.

 note
**Security** **alert**

As we mentioned previously, switching SELinux to **Permissive** mode or
completely disabling it is *not* *good* *practice,* as it opens you up
to potential security threats.[]{.sentence-end} Instead of doing that,
users should create custom policies to manage the necessary exceptions.


By default, SELinux uses[]{#Chapter_10.xhtml#idx_19433bd5 .index-entry
index-entry="targeted policy type"} a **targeted** policy type, which
tries to target and confine []{#Chapter_10.xhtml#idx_9d4d6977
.index-entry index-entry="SELinux interaction:with containers"}specific
object types (processes, files, devices, and so on)
using[]{#Chapter_10.xhtml#idx_1905e269 .index-entry
index-entry="containers:SELinux interaction"} a set of predefined
policies.

SELinux allows different kinds of access control.[]{.sentence-end} They
can be summarized as follows:

-   **Type** **Enforcement** (**TE**): This controls access to resources
    according to process and file types.[]{.sentence-end} This is
    the[]{#Chapter_10.xhtml#idx_df1f2ef5 .index-entry
    index-entry="access control, SELinux:Type Enforcement (TE)"} main
    use case of SELinux access control.
-   **Role-Based** **Access** **Control** (**RBAC**): This controls
    access to resources using SELinux []{#Chapter_10.xhtml#idx_60e9fc60
    .index-entry
    index-entry="access control, SELinux:Role-Based Access Control (RBAC)"}users
    (which can be mapped to real system users) and their associated
    SELinux roles.
-   **Multi-Level** **Security** (**MLS**): This grants all processes
    with the same sensitivity level read/write access
    to[]{#Chapter_10.xhtml#idx_b935c1be .index-entry
    index-entry="access control, SELinux:Multi-Level Security (MLS)"}
    the resources.
-   **Multi-Category** **Security** (**MCS**):
    This[]{#Chapter_10.xhtml#idx_8803eced .index-entry
    index-entry="access control, SELinux:Multi-Category Security (MCS)"}
    controls access using **categories**, which are plain text labels
    that are applied to resources.[]{.sentence-end} Categories
    []{#Chapter_10.xhtml#idx_27f2ab24 .index-entry
    index-entry="categories"}are used to create compartments of objects,
    along with the other SELinux labels.[]{.sentence-end} Only processes
    that belong to the same category can access a given
    resource.[]{.sentence-end} In *Chapter* *5*, *Implementing*
    *Storage* *for* *the* *Container\'s* *Data*, we discussed MCS and
    how we can map categories to resources that have been accessed by
    containers.

With type[]{#Chapter_10.xhtml#idx_f0c6c3d1 .index-entry
index-entry="SELinux interaction:with containers"} enforcement, the
system files receive labels[]{#Chapter_10.xhtml#idx_79f3d891
.index-entry index-entry="types"} called **types**, while processes
receive labels[]{#Chapter_10.xhtml#idx_5a503819 .index-entry
index-entry="domains"} called **domains**.[]{.sentence-end} A process
that belongs to a domain can be allowed to access a file that belongs to
a given type, and this access can be audited by SELinux.

For example, according []{#Chapter_10.xhtml#idx_9ad34420 .index-entry
index-entry="containers:SELinux interaction"}to SELinux, the Apache
`httpd`{.inlineCode} process, which is labeled with the
`httpd_t`{.inlineCode} domain, can access files or directories with
`httpd_sys_content_t`{.inlineCode} labels.

An SELinux-type policy is based on the following pattern:

``` {.programlisting .snippet-code}
POLICY DOMAIN TYPE:CLASS OPERATION;
```

Here, `POLICY`{.inlineCode} is the kind of policy (`allow`{.inlineCode},
`allowxperm`{.inlineCode}, `auditallow`{.inlineCode},
`neverallow`{.inlineCode}, `dontaudit`{.inlineCode}, and so on),
`DOMAIN`{.inlineCode} is the process domain, `TYPE`{.inlineCode} is the
resource type context, `CLASS`{.inlineCode} is the object category (for
example, `file`{.inlineCode}, `dir`{.inlineCode},
`lnk_file`{.inlineCode}, `chr_file`{.inlineCode},
`blk_file`{.inlineCode}, `sock_file`{.inlineCode}, or
`fifo_file`{.inlineCode}), and `OPERATION`{.inlineCode} is a list of
actions that are handled by the policy (for example,
`open`{.inlineCode}, `read`{.inlineCode}, `use`{.inlineCode},
`lock`{.inlineCode}, `getattr`{.inlineCode}, or `revc`{.inlineCode}).

The following example shows a basic `allow`{.inlineCode} rule:

``` {.programlisting .snippet-code}
allow myapp_t myapp_log_t:file { read_file_perms append_file_perms };
```

In this example, the[]{#Chapter_10.xhtml#idx_463e675d .index-entry
index-entry="SELinux interaction:with containers"} process that\'s
running in the `myapp_t`{.inlineCode} domain is allowed to access files
of the `myapp_log_t`{.inlineCode} type and perform the
`read_file_perms`{.inlineCode} and `append_file_perms`{.inlineCode}
actions.

SELinux manages policies in a modular fashion, allowing users to
dynamically load and unload policy modules without the need to recompile
the whole policy set every time.[]{.sentence-end} Policies can be loaded
and unloaded using the `semodule`{.inlineCode} utility, as shown in the
following example, which shows an example of loading a custom policy:

``` {.programlisting .snippet-con}
# semodule -i custompolicy.pp
```

The `semodule`{.inlineCode} utility can also be used to view all the
loaded policies:

``` {.programlisting .snippet-con}
# semodule -l
```

On Fedora, CentOS, RHEL, and []{#Chapter_10.xhtml#idx_5b5dc7b7
.index-entry index-entry="containers:SELinux interaction"}derivative
distributions, the current binary policy is installed under the
`/etc/selinux/targeted/policy`{.inlineCode} directory in a file named
`polixy.XX`{.inlineCode}, with `XX`{.inlineCode} representing the policy
version.

On the same distributions, container policies are defined inside the
`container-selinux`{.inlineCode} package, which contains the already
compiled SELinux module.[]{.sentence-end} The source code of the package
is available on GitHub if you wish to look at it in more detail:
[[https://github.com/containers/container-selinux]{.url}](https://github.com/containers/container-selinux){style="text-decoration: none;"}.

By looking at the repository\'s content, we will find the three most
important policy source files for developing any module:

-   `container.fc`{.inlineCode}: This file defines the files and
    directories that are bound to the types defined in the module.
-   `container.te`{.inlineCode}: This file defines the policy rules,
    attributes, and aliases.
-   `container.if`{.inlineCode}: This file defines the module
    interface.[]{.sentence-end} It contains a set of public macro
    functions that are exposed by the module.

A process that\'s running inside a container is labeled with the
`container_t`{.inlineCode} domain.[]{.sentence-end} It has read/write
access to resources labeled with the `container_file_t`{.inlineCode}
type context and read/execute access to resources labeled with the
`container_share_t`{.inlineCode} type context.

When a container is executed, the `podman`{.inlineCode} process, as well
as the container runtime and the `conmon`{.inlineCode} process, run with
the `container_runtime_t`{.inlineCode} domain type and are allowed to
execute processes that []{#Chapter_10.xhtml#idx_2b4cf7f4 .index-entry
index-entry="SELinux interaction:with containers"}transition only to
specific types.[]{.sentence-end} Those types are
grouped[]{#Chapter_10.xhtml#idx_7fe2566d .index-entry
index-entry="containers:SELinux interaction"} in the
`container_domain`{.inlineCode} attribute and can be inspected with the
`seinfo`{.inlineCode} utility (installed with the
`setools-console`{.inlineCode} package on Fedora), as shown in the
following code:

``` {.programlisting .snippet-con}
$ seinfo -a container_domain -x
Type Attributes: 1
  attribute container_domain;
    container_device_plugin_init_t
    container_device_plugin_t
    container_device_t
    container_engine_t
    container_init_t
    container_kvm_t
    container_logreader_t
    container_logwriter_t
    container_t
    container_userns_t
```

The `container_domain`{.inlineCode} attribute is declared in the
`container.te`{.inlineCode} source file in the
`container-policy`{.inlineCode} repository using the
`attribute`{.inlineCode} keyword:

``` {.programlisting .snippet-code}
attribute container_domain;
attribute container_user_domain;
attribute container_net_domain;
```

The preceding attributes are mapped to the `container_t`{.inlineCode}
type using a `typeattribute`{.inlineCode} declaration:

``` {.programlisting .snippet-code}
typeattribute container_t container_domain, container_net_domain, container_user_domain;
```

Using this approach, SELinux guarantees process isolation across
containers and between a container and its host.[]{.sentence-end} In
this way, a process escaping the container (maybe exploiting a
vulnerability) cannot access resources on the host or inside other
containers.

When a container is[]{#Chapter_10.xhtml#idx_f2660d37 .index-entry
index-entry="SELinux interaction:with containers"} created, the image\'s
read-only layers, which form[]{#Chapter_10.xhtml#idx_0f7abb49
.index-entry index-entry="containers:SELinux interaction"} the
`OverlayFS`{.inlineCode} set of `LowerDirs`{.inlineCode}, are labeled
with the `container_ro_file_t`{.inlineCode} type, which prevents the
container from writing inside those directories.[]{.sentence-end} At the
same time, `MergedDir`{.inlineCode}, which is the sum of
`LowerDirs`{.inlineCode} and `UpperDir`{.inlineCode}, is writable and
labeled as `container_file_t`{.inlineCode}.

To prove this, let\'s run a **rootful**
container[]{#Chapter_10.xhtml#idx_894bbad6 .index-entry
index-entry="rootful container"} with the `c1`{.inlineCode} and
`c2`{.inlineCode} MCS categories:

``` {.programlisting .snippet-con}
# podman run -d --name selinux_test1 --security-opt label=level:s0:c1,c2 nginx
```

Now, we can find all the files labeled as
`container_file_t:s0:c1`{.inlineCode}`,c2`{.inlineCode} under the host
filesystem:

``` {.programlisting .snippet-con}
#
# find /var/lib/containers/storage/overlay -type f -context '*container_file_t:s0:c1,c2*' -printf '%-50Z%p\n'
system_u:object_r:container_file_t:s0:c1,c2       /var/lib/containers/storage/overlay/4b147975bb5c336b10e71d21c49fe88ddb00d0569b77ddab1d7737f80056677b/merged/lib/x86_64-linux-gnu/libreadline.so.8.1
system_u:object_r:container_file_t:s0:c1,c2       /var/lib/containers/storage/overlay/4b147975bb5c336b10e71d21c49fe88ddb00d0569b77ddab1d7737f80056677b/merged/lib/x86_64-linux-gnu/libhistory.so.8.1
system_u:object_r:container_file_t:s0:c1,c2       /var/lib/containers/storage/overlay/4b147975bb5c336b10e71d21c49fe88ddb00d0569b77ddab1d7737f80056677b/merged/lib/x86_64-linux-gnu/libexpat.so.1.6.12
system_u:object_r:container_file_t:s0:c1,c2       /var/lib/containers/storage/overlay/4b147975bb5c336b10e71d21c49fe88ddb00d0569b77ddab1d7737f80056677b/merged/lib/udev/rules.d/96-e2scrub.rules
system_u:object_r:container_file_t:s0:c1,c2       /var/lib/containers/storage/overlay/4b147975bb5c336b10e71d21c49fe88ddb00d0569b77ddab1d7737f80056677b/merged/lib/terminfo/r/rxvt-unicode-256color
system_u:object_r:container_file_t:s0:c1,c2       /var/lib/containers/storage/overlay/4b147975bb5c336b10e71d21c49fe88ddb00d0569b77ddab1d7737f80056677b/merged/lib/terminfo/r/rxvt-unicode
[…output omitted...]
```

As expected, the `container_file_t`{.inlineCode} label, which is
associated with the `c1`{.inlineCode} and `c2`{.inlineCode} categories,
is applied to all[]{#Chapter_10.xhtml#idx_724921fb .index-entry
index-entry="SELinux interaction:with containers"} the files under the
`MergedDir`{.inlineCode} container.

At the same time, we []{#Chapter_10.xhtml#idx_e27f8a07 .index-entry
index-entry="containers:SELinux interaction"}can demonstrate that the
container\'s `LowerDirs`{.inlineCode} are labeled as
`container_ro_file_t`{.inlineCode}.[]{.sentence-end} First, we need to
extract the container\'s `LowerDirs`{.inlineCode} list:

``` {.programlisting .snippet-con}
# podman inspect selinux_test1 \
  --format '{{.GraphDriver.Data.LowerDir}}'
/var/lib/containers/storage/overlay/9566cbcf1773eac59951c14c52156a6164db1b0d8026d015e193774029db18a5/diff:/var/lib/containers/storage/overlay/24de59cced7931bbcc0c4a34d4369c15119a0b8b180f98a0434fa76a6dfcd490/diff:/var/lib/containers/storage/overlay/1bb84245b98b7e861c91ed4319972ed3287bdd2ef02a8657c696a76621854f3b/diff:/var/lib/containers/storage/overlay/97f26271fef21bda129ac431b5f0faa03ae0b2b50bda6af969315308fc16735b/diff:/var/lib/containers/storage/overlay/768ef71c8c91e4df0aa1caf96764ceec999d7eb0aa584e241246815c1fa85435/diff:/var/lib/containers/storage/overlay/2edcec3590a4ec7f40cf0743c15d78fb39d8326bc029073b41ef9727da6c851f/diff
```

The rightmost directory represents the container\'s lowest layer and is
usually the base filesystem tree of the image.[]{.sentence-end} Let\'s
inspect the type context of this directory:

``` {.programlisting .snippet-con}
# ls -alZ /var/lib/containers/storage/overlay/2edcec3590a4ec7f40cf0743c15d78fb39d8326bc029073b41ef9727da6c851f/diff
total 84
dr-xr-xr-x. 21 root root unconfined_u:object_r:container_ro_file_t:s0 4096 Jan  5 23:16 .
drwx------.  6 root root unconfined_u:object_r:container_ro_file_t:s0 4096 Jan  5 23:16 ..
drwxr-xr-x.  2 root root unconfined_u:object_r:container_ro_file_t:s0 4096 Dec 20 00:00 bin
drwxr-xr-x.  2 root root unconfined_u:object_r:container_ro_file_t:s0 4096 Dec 11 17:25 boot
drwxr-xr-x.  2 root root unconfined_u:object_r:container_ro_file_t:s0 4096 Dec 20 00:00 dev
drwxr-xr-x. 30 root root unconfined_u:object_r:container_ro_file_t:s0 4096 Dec 20 00:00 etc
drwxr-xr-x.  2 root root unconfined_u:object_r:container_ro_file_t:s0 4096 Dec 11 17:25 home
drwxr-xr-x.  8 root root unconfined_u:object_r:container_ro_file_t:s0 4096 Dec 20 00:00 lib
[...omitted output...]
```

The preceding output also []{#Chapter_10.xhtml#idx_afb7d5a1 .index-entry
index-entry="containers:SELinux interaction"}shows another interesting
aspect: since the `LowerDir`{.inlineCode}
layers[]{#Chapter_10.xhtml#idx_e41318cd .index-entry
index-entry="SELinux interaction:with containers"} are shared across
multiple containers that use the same image, we won\'t find any MCS
categories that have been applied here.

Containers do not have read/write access to files or directories that
are not labeled as `container_file_t`{.inlineCode}.[]{.sentence-end}
Previously, we saw that it is possible to relabel those files by
applying the `:z`{.inlineCode} suffix to mounted volumes or by manually
relabeling them in advance before running the containers.

However, relabeling []{#Chapter_10.xhtml#idx_426d60ac .index-entry
index-entry="SELinux interaction:with containers"}crucial directories
such as `/home`{.inlineCode} or `/var/logs`{.inlineCode} is
a[]{#Chapter_10.xhtml#idx_fe9ae000 .index-entry
index-entry="containers:SELinux interaction"} very bad idea since many
other non-containerized processes won\'t be able to access them anymore.

The only solution is to manually create custom policies that override
the default behavior.[]{.sentence-end} However, this is too complex to
manage in everyday use and production environments.

Luckily, we can solve this limitation with a tool that generates custom
SELinux security profiles for our []{#Chapter_10.xhtml#idx_083dc56c
.index-entry index-entry="Udica"}containers: **Udica**.

## Introducing Udica {#Chapter_10.xhtml#h2_267 .heading-2}

Udica is an []{#Chapter_10.xhtml#idx_2886b87d .index-entry
index-entry="Udica"}open source project
([[https://github.com/containers/udica]{.url}](https://github.com/containers/udica){style="text-decoration: none;"})
that was created by Lukas Vrabec, SELinux evangelist and team leader of
the SELinux and Security Special Projects engineering teams at Red Hat.

Udica aims to overcome the rigid policy limitations that were described
previously by generating SELinux profiles for containers and allowing
them to access resources that would normally be prevented with the
common `container_t`{.inlineCode} domain.

To install Udica on Fedora, simply run the following command:

``` {.programlisting .snippet-con}
$ sudo dnf install -y udica setools-console container-selinux
```

On other distributions, Udica can be installed from its source by
running the following commands:

``` {.programlisting .snippet-con}
$ sudo dnf install -y setools-console git container-selinux
$ git clone
$ cd udica && sudo python3 ./setup.py install
```

To demonstrate how Udica works, we are going to create a container that
writes to the `/var/log`{.inlineCode} directory of the host, which is
bind-mounted when the container is created.[]{.sentence-end} By default,
the process with the `container_t`{.inlineCode} domain would not be able
to write a directory labeled with the `var_log_t`{.inlineCode} type.

The following script, which has been executed inside the container, is
an endless loop that writes a log line composed of the current date and
a counter:

`Chapter10/custom_logger/logger.sh`{.inlineCode}

``` {.programlisting .snippet-code}
#!/bin/bash
set -euo pipefail
trap "echo Exited; exit;" SIGINT SIGTERM

# Run an endless loop writing a simple log entry with date
count=1
while true; do
echo "$(date +%y/%m/%d_%H:%M:%S) - Line #$count" | tee -a /var/log/custom.log
  count=$((count+1))
  sleep 2
done
```

The preceding script uses the
`set`{.inlineCode}` `{.inlineCode}`-euo`{.inlineCode}` `{.inlineCode}`pipefail`{.inlineCode}
option to exit immediately in case an error occurs, and the
`tee`{.inlineCode} utility to write the command\'s output both to
standard output and the `/var/log/custom.log`{.inlineCode} file in
append mode.[]{.sentence-end} The `count`{.inlineCode} variable
increments on each loop cycle.

The Dockerfile for this []{#Chapter_10.xhtml#idx_d0932e8e .index-entry
index-entry="Udica"}container is kept minimal -- it just copies the
logger script and executes it at container startup:

`Chapter10/custom_logger/Dockerfile`{.inlineCode}

``` {.programlisting .snippet-code}
FROM docker.io/library/fedora
# Copy the logger.sh script
COPY --chmod=755 logger.sh /
# Exec the logger.sh script
CMD ["/logger.sh"]
```

 note
**Important**

The `logger.sh`{.inlineCode} script is copied with the
`--chmod=755`{.inlineCode} option to set up the proper executable
permissions so that it can be launched correctly at container startup.


The container image is built with the name `custom_logger`{.inlineCode}:

``` {.programlisting .snippet-con}
# cd /Chapter10/custom_logger
# buildah build -t custom_logger .
```

Now, it\'s time to test the container and see how it
behaves.[]{.sentence-end} The `/var/log`{.inlineCode} directory is
bind-mounted with `rw`{.inlineCode} permissions to the container path
`/var/log`{.inlineCode}, without this altering its type
context.[]{.sentence-end} We should keep the execution in the foreground
to see the immediate output:

``` {.programlisting .snippet-con}
# podman run -v /var/log:/var/log:rw \
  --name custom_logger1 custom_logger
tee: /var/log/custom.log: Permission denied
22/01/08_09:09:33 - Custom log event #1
```

As expected, the script failed to write to the target
file.[]{.sentence-end} We could fix this by changing the directory type
context to `container_file_t`{.inlineCode}, but, as we learned
previously, this is a poor idea since it would prevent other processes
from writing their logs.

Instead, we can use []{#Chapter_10.xhtml#idx_c0d3cd2d .index-entry
index-entry="Udica"}Udica to generate a custom SELinux security profile
for the container.[]{.sentence-end} In the following code, the container
specs are exported to a `container.json`{.inlineCode} file and then
parsed by Udica to generate a custom profile called
`custom_logger`{.inlineCode}:

``` {.programlisting .snippet-con}
# podman inspect custom_logger1 > container.json
# udica -j container.json custom_logger
Policy custom_logger created!

Please load these modules using:
# semodule -i custom_logger.cil /usr/share/udica/templates/{base_container.cil,log_container.cil}
Restart the container with: "--security-opt label=type:custom_logger.process" parameter
```

Once the profile has been generated, Udica outputs the instructions to
configure the container.[]{.sentence-end} First, we need to load the new
custom policy using the `semodule`{.inlineCode}
utility.[]{.sentence-end} The generated[]{#Chapter_10.xhtml#idx_9abeb5ea
.index-entry index-entry="Common Intermediate Language (CIL)"} file is
in **Common** **Intermediate** **Language** (**CIL**) format, an
intermediate policy language for SELinux.[]{.sentence-end} Along with
the generated CIL file, the example loads some Udica templates,
`/usr/share/udica/templates/base_container.cil`{.inlineCode} and
`/usr/share/udica/templates/log_container.cil`{.inlineCode}, whose rules
are inherited in the custom container policy file.

Let\'s load the modules using the suggested command:

``` {.programlisting .snippet-con}
# semodule -i custom_logger.cil /usr/share/udica/templates/{base_container.cil,log_container.cil}
```

After loading the modules[]{#Chapter_10.xhtml#idx_5ef194a7 .index-entry
index-entry="Udica"} in SELinux, we are ready to run the container with
the custom `custom_logger.process`{.inlineCode} label, which is passed
as an argument to Podman\'s `--security-opt`{.inlineCode} option.

All the other container options were kept identical, except for its
name, which has been updated to `custom_logger2`{.inlineCode} to
differentiate it from the previous instance:

``` {.programlisting .snippet-con}
# podman run -v /var/log:/var/log:rw \
  --name custom_logger2 \
  --security-opt label=type:custom_logger.process \
  custom_logger
22/01/08_09:05:19 - Line #1
22/01/08_09:05:21 - Line #2
22/01/08_09:05:23 - Line #3
22/01/08_09:05:25 - Line #5
[...Omitted output...]
```

This time, the script successfully wrote to the
`/var/log/custom.log`{.inlineCode} file thanks to the custom profile
that was generated with Udica.

Note that the container processes are not running with the
`container_t`{.inlineCode} domain, but with the new
`custom_logger.process`{.inlineCode} superset, which includes additional
rules on top of the default.

We can confirm this by running the following command on the host:

``` {.programlisting .snippet-con}
# ps auxZ | grep 'custom_logger.process'
unconfined_u:system_r:container_runtime_t:s0-s0:c0.c1023 root 26546 0.1  0.6 1365088 53768 pts/0 Sl+ 09:16   0:00 podman run -v /var/log:/var/log:rw --security-opt label=type:custom_logger.process custom_logger system_u:system_r:custom_logger.process:s0:c159,c258 root 26633 0.0  0.0 4180 3136 ? Ss 09:16   0:00 /bin/bash /logger.sh
system_u:system_r:custom_logger.process:s0:c159,c258 root 26881 0.0  0.0 2640 1104 ? S 09:18   0:00 sleep 2
```

Udica creates[]{#Chapter_10.xhtml#idx_6c429514 .index-entry
index-entry="Udica"} the custom policy by parsing the JSON spec file and
looking for the container mount points, ports, and
capabilities.[]{.sentence-end} Let\'s look at the content of the
generated `custom_logger.cil`{.inlineCode} file from our example:

``` {.programlisting .snippet-code}
(block custom_logger
    (blockinherit container)
    (allow process process ( capability ( chown dac_override fowner fsetid kill net_bind_service setfcap setgid setpcap setuid sys_chroot )))

    (blockinherit log_rw_container)
```

The CIL language syntax is beyond the scope of this book, but we can
still notice some interesting things:

-   The `custom_logger`{.inlineCode} profile is defined by a
    `block`{.inlineCode} statement
-   The `allow`{.inlineCode} rule enables the default capabilities for
    the container
-   The policy inherits the `container`{.inlineCode} and
    `log_rw_container`{.inlineCode} blocks with the
    `blockinherit`{.inlineCode} statements

The generated CIL file inherits the blocks that have been defined in the
available Udica templates, each one focused on specific
actions.[]{.sentence-end} On Fedora, the templates are installed via the
`container-selinux`{.inlineCode} package and are available in the
`/usr/share/udica/templates/`{.inlineCode} folder:

``` {.programlisting .snippet-con}
# ls -1 /usr/share/udica/templates/
base_container.cil
config_container.cil
home_container.cil
log_container.cil
net_container.cil
tmp_container.cil
tty_container.cil
virt_container.cil
x_container.cil
```

The available templates are implemented for common scenarios, such as
accessing log directories or user homes, or even opening network
ports.[]{.sentence-end} Among them, the
`base_container.cil`{.inlineCode} template is always included by all the
Udica-generated policies as the base building block that\'s used to
generate the custom policies.

According to the behavior of the container that\'s derived from the spec
file, other templates are included.[]{.sentence-end} For example, the
policy inherits the `log_rw_container`{.inlineCode} block from the
`log_container.cil`{.inlineCode} template to let the custom logger
container access the `/var/log`{.inlineCode} directory.

Udica is a []{#Chapter_10.xhtml#idx_ce3668a4 .index-entry
index-entry="Udica"}great tool for addressing container isolation issues
and helps administrators address SELinux confinement use cases by
overcoming the complexity of writing rules manually.

Generated security profiles can also be versioned inside a GitHub
repository and reused for similar containers on different hosts.

# Summary {#Chapter_10.xhtml#h1_268 .heading-1}

In this chapter, we learned how to develop and apply techniques to
improve the overall security of our container-based service
architecture.[]{.sentence-end} We learned how leveraging rootless
containers and avoiding UID 0 can reduce the attack surface of our
services.[]{.sentence-end} Then, we learned how to sign and trust
container images to avoid MITM attacks.[]{.sentence-end} Finally, we
went under the hood of containers\' tools and looked at the Linux
kernel\'s capabilities and the SELinux subsystem, which can help us
fine-tune various security aspects for our running containers.

Now that we\'ve done a deep dive into security, we are ready to move on
to the next chapter, where we will take an advanced look at networking
for containers.

# Further reading {#Chapter_10.xhtml#h1_269 .heading-1}

For more information about the topics that were covered in this chapter,
take a look at the following resources:

-   MITRE ATT&CK Container Matrix:
    [[https://attack.mitre.org/matrices/enterprise/containers/]{.url}](https://attack.mitre.org/matrices/enterprise/containers/){style="text-decoration: none;"}
-   Sigstore project home page:
    [[https://docs.sigstore.dev/]{.url}](https://docs.sigstore.dev/){style="text-decoration: none;"}
-   Podman image signing tutorial:
    [[https://github.com/containers/podman/blob/main/docs/tutorials/image_signing.md]{.url}](https://github.com/containers/podman/blob/main/docs/tutorials/image_signing.md){style="text-decoration: none;"}
-   Lukas Vrabec\'s blog:
    [[https://lukas-vrabec.com/]{.url}](https://lukas-vrabec.com/){style="text-decoration: none;"}
-   CIL introduction and design principles:
    [[https://github.com/SELinuxProject/cl/wiki]{.url}](https://github.com/SELinuxProject/cl/wiki){style="text-decoration: none;"}
-   Udica introduction on Red Hat\'s blog:
    [[https://www.redhat.com/en/blog/generate-selinux-policies-containers-with-udica]{.url}](https://www.redhat.com/en/blog/generate-selinux-policies-containers-with-udica){style="text-decoration: none;"}

# Join us on Discord {#Chapter_10.xhtml#h1_270 .heading-1}

For discussions around the book and to connect with your peers, join us
on Discord at
[[packt.link/discordcloud]{.url}](https://packt.link/discordcloud){style="text-decoration: none;"}
or scan the QR code below:

![Image](images/B31467_10_1.png){style="width:25%;"}


[]{#Part_3.xhtml}

 {.section .part}
