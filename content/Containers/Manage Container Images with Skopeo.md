## Managing container images with Skopeo 

So far, we have learned about many container registry concepts, including the differences between private and public registries, their compliance with OCI image specifications, and how to consume images with Podman and Buildah to build and run containers.

However, sometimes we need to implement simple image manipulation tasks such as moving an image from a registry to a mirror, inspecting a remote image without the need to pull it locally, or even signing images.

The community that gave birth to Podman and Buildah developed a third amazing tool, **Skopeo** (https://github.com/containers/skopeo), which exactly implements the features described previously.

Skopeo was designed as an image and registry manipulation tool for DevOps teams and is not intended to run containers (the main role of Podman) nor build OCI images (the main role of Buildah). Instead, it offers a minimal and straightforward command-line interface with basic image manipulation commands that will prove to be extremely useful in different contexts.

Let\'s inspect the most interesting features in the next subsections.

### Installing Skopeo 

Skopeo is a Go binary tool that is already packaged and available for many distributions. It can also be built and installed from source directly.

This section provides a non-exhaustive list of installation examples on the major distributions. For the sake of clarity, it is important to reiterate that the book lab environments were all based on Fedora 40:

- **Fedora,** **RHEL 8/9, CentOS 8,** **and CentOS Stream 8/9**: To install Skopeo on RHEL-like systems, run the following `dnf` command:

 ```$ sudo dnf -y install skopeo ```

- **Debian**: To install Skopeo on Debian Bullseye, Testing, and Unstable (Sid), run the following `apt-get` commands:

 ```$ sudo apt-get update $ sudo apt-get -y install skopeo ```

- **Ubuntu**: To install Skopeo on Ubuntu 20.10 and newer, run the following command:

 ```$ sudo apt-get -y update $ sudo apt-get -y install skopeo ```

- **Arch Linux**: To install Skopeo on Arch Linux, run the following `pacman` command:

 ```$ sudo pacman -S skopeo ```

- **openSUSE**: To install Skopeo on openSUSE, run the following `zypper` command:

 ``` {.programlisting .snippet-con-one} $ sudo zypper install skopeo ```

- **macOS**: To install Skopeo on macOS, run the following `brew` command:

 ``` {.programlisting .snippet-con-one} $ brew install skopeo ```

- **Building from source**: Skopeo can also be built from source. As for Buildah, for the purposes of this book, we will keep the focus on simple deployment methods, but if you\'re curious, you can find a dedicated *install* section in the main project repository that illustrates how to build Skopeo from source: https://github.com/containers/skopeo/blob/main/install.md#building-from-source.

 The preceding link shows examples of containerized and non-containerized builds.

- **Running Skopeo in a container**: Skopeo is also released as a container image that can be executed with Podman. To pull and run the latest version of Skopeo as a container, use the following `podman` command:

 ``` {.programlisting .snippet-con-one} $ podman run quay.io/skopeo/stable:latest <command> <options> ```

- **Windows**: At the time of writing this book, there is no build available for Microsoft Windows. However, you can install it in **Windows Linux Subsystem** (**WSL**). Since WSL is essentially a Linux environment, the installation process for Skopeo depends entirely on which distribution you have installed (Fedora, Ubuntu, and so on).

Skopeo uses the same system and local configuration files described for Podman and Buildah; therefore, we can immediately focus on the installation verification and the analysis of the most common use cases.

### Verifying the installation 

To verify the correct installation, simply run the `skopeo` command with the `-h` or `--help` option to view all available commands, as in the following example:

``` $ skopeo -h ```

The expected output will show, among the utility options, all the available commands, each one with a description of the command scope. The full list of commands is as follows:

- `copy`: Copy an image across locations, using different transports, such as the Docker Registry, local directories, OCI, tarballs, OSTree, and OCI archives
- `delete`: Delete an image from a target location
- `generate-sigstore-key`: Generate a `sigstore` public/private key pair
- `help`: Print `help` commands
- `inspect`: Inspect the metadata, tags, and configuration of an image in a target location
- `list-tags`: Show the available tags for a specific image repository
- `login`: Authenticate to a remote registry
- `logout`: Log out from a remote registry
- `manifest-digest`: Produce a manifest digest for a file
- `standalone-sign`: A debugging tool to publish and sign an image using local files
- `standalone-verify`: Verify an image signature using local files
- `sync`: Synchronize one or more images across locations

Let\'s now inspect in greater detail some of the most interesting Skopeo commands.

### Copying images across locations 

Podman, just like Docker, can be used not only to run containers but also to pull images locally and push them to other locations. However, one of the main caveats is the need to run two commands, one to pull and one to push, while the local image store remains filled with the pulled images. Therefore, users should periodically clean up the local store.

Skopeo offers a smarter and simpler way to achieve this goal with the `skopeo copy` command. The command implements the following syntax:

``` skopeo copy [command options] SOURCE-IMAGE DESTINATION-IMAGE ```

In this generic description, `SOURCE-IMAGE` and `DESTINATION-IMAGE` are images belonging to local or remote locations and reachable using one of the following transports:

- `docker://docker-reference`: This transport is related to images stored in registries implementing the *Docker Registry HTTP API V2*.

 This setting uses the `/etc/containers/registries.conf` or `$HOME/.config/containers/registries.conf` file to obtain further registry configurations.

 The `docker-reference` field follows the format `name[:tag|@digest]`.

- `containers-storage:[[storage-specifier]]{image-id|docker-reference[@image-id]}`: This setting refers to an image in local container storage.

 The `storage-specifier` field is in the format `[[driver@]root[+run-root][:options]]`.

- `dir:path`: This setting refers to an existing local directory that holds manifests, layers (in tarball format), and signatures

- `docker-archive:path[:{docker-reference|@source-index}]`: This setting refers to a Docker archive obtained with the `docker save` or `podman save` command

- `docker-daemon:docker-reference|algo:digest`: This setting refers to image storage in the Docker daemon\'s internal storage

- `oci:path[:tag]`: This setting refers to an image stored in a local path compliant with the OCI layout specifications

- `oci-archive:path[:tag]`: This setting refers to an OCI layout specification-compliant image stored in tarball format

Let\'s inspect some usage examples of the `skopeo copy` command in real-world scenarios. The first example shows how to copy an image from a remote registry to another remote registry:

``` $ skopeo copy \ docker://docker.io/library/nginx:latest \ docker://private-registry.example.com/lab/nginx:latest ```

The preceding example does not take care of registry authentication, which is usually a requirement to push images to the remote repository. In the next example, we show a variant where both source and target registry are decorated with authentication options:

``` $ skopeo copy \ --src-creds USERNAME:PASSWORD \ --dest-creds USERNAME:PASSWORD \ docker://registry1.example.com/mirror/nginx:latest \ docker://registry2.example.com/lab/nginx:latest ```

The previous approach, despite working perfectly, has the limitation of passing username and password strings as clear-text strings. To avoid this, we can use the `skopeo login` command to authenticate to our registries before running `skopeo copy`.

The third example shows a pre-authentication to the destination registry, assuming that the source registry is publicly accessible for pulls:

``` $ skopeo login private-registry.example.com $ skopeo copy \ docker://docker.io/library/nginx:latest \ docker://private-registry.example.com/lab/nginx:latest ```

When we log in to the source/target registries, the system persists the registry-provided auth tokens in dedicated auth files that we can reuse later for further access.

By default, Skopeo looks at the `${XDG_RUNTIME_DIR}/containers/auth.json` path, but we can provide a custom location for the auth file. For example, if we used the Docker container runtime before, we could find it in the `${HOME}/.docker/config.json` path. This file contains a simple JSON object that holds, for every used registry, the token obtained upon authentication. The client (Podman, Skopeo, or Buildah) will use this token to directly access the registry. Basically, once one tool logs in, all tools can use the token. `podman login` allows `skopeo copy` and `buildah push` to access the registry, for example.

The following example shows the usage of the auth file, provided with a custom path:

``` $ skopeo copy \ --authfile ${HOME}/.docker/config.json \ docker://docker.io/library/nginx:latest \ docker://private-registry.example.com/lab/nginx:latest ```

Another common issue that can be encountered when working with a private registry is the lack of certificates signed by a known **certification authority** (**CA**) or the lack of HTTPS communication (which means that all traffic is completely unencrypted). If we consider these totally non-secure scenarios safe to trust in a lab environment, we can skip the TLS verification with the `--dest-tls-verify` and `--src-tls-verify` options, which accept a simple Boolean value.

The following example shows how to skip the TLS verification on the target registry:

``` $ skopeo copy \ --authfile ${HOME}/.docker/config.json \ --dest-tls-verify false \ docker://docker.io/library/nginx:latest \ docker://private-registry.example.com/lab/nginx:latest ```

So far, we\'ve seen how to move images across public and private registries, but we can use Skopeo to move images to and from local stores easily. For example, we can use Skopeo as a highly specialized push/pull tool for images inside our build pipelines.

The next example shows how to push a locally built image to a public registry. The image already exists locally and is then pushed to the remote registry:

``` $ podman images REPOSITORY TAG IMAGE ID CREATED SIZE <namespace>/python_httpd latest 4067fa24786a 12 days ago 318 MB $ skopeo copy \ --authfile ${HOME}/.docker/config.json \ containers-storage:quay.io/<namespace>/python_httpd \ docker://quay.io/<namespace>/python_httpd:latest ```

This is an amazing way to manage an image push with total control over the push/pull process and shows how the three tools
-- Podman, Buildah, and Skopeo -- can fulfill specialized tasks in our DevOps environment, each one accomplishing the purpose it was designed for at its best.

Let\'s see another example, this time showing how to pull an image from a remote registry to an OCI-compliant local store:

``` $ skopeo copy \ --authfile ${HOME}/.docker/config.json \ docker://docker.io/library/nginx:latest \ oci:/tmp/nginx ```

The output folder is compliant with the OCI image specifications and will have the following structure (blob hashes cut for layout reasons). This basically expands the image tarball at a specified location, allowing for easy inspection:

``` $ tree /tmp/nginx /tmp/nginx/ ├─ blobs │ └─sha256 │ ├──21e0df283cd68384e5e8dff7e6be1774c86ea3110c1b1e932[...] │ ├──44be98c0fab60b6cef9887dbad59e69139cab789304964a19[...] │ ├──77700c52c9695053293be96f9cbcf42c91c5e097daa382933[...] │ ├──81d15e9a49818539edb3116c72fbad1df1241088116a7363a[...] │ ├──881ff011f1c9c14982afc6e95ae70c25e38809843bb7d42ab[...] │ ├──d86da3a6c06fb46bc76d6dc7b591e87a73cb456c990d814fd[...] │ ├──e5ae68f740265288a4888db98d2999a638fdcb6d725f42767[...] │ └──ed835de16acd8f5821cf3f3ef77a66922510ee6349730d89a[...] ├─ index.json └─ oci-layout ```

The files inside the `blobs/sha256` folder include the image manifest (in JSON format) and the image layers in compressed tarball format

It\'s interesting to know that Podman can seamlessly run a container based on a local folder compliant with the OCI image specifications. The next example shows how to run an NGINX container from the previously downloaded image:

``` $ podman run -d oci:/tmp/nginx Getting image source signatures Copying blob e5ae68f74026 done Copying blob 21e0df283cd6 done Copying blob ed835de16acd done Copying blob 881ff011f1c9 done Copying blob 77700c52c969 done Copying blob 44be98c0fab6 done Copying config 81d15e9a49 done Writing manifest to image destination Storing signatures 90493fe89f024cfffda3f626acb5ba8735cadd827be6c26fa44971108e09b54f ```

Notice the `oci:` prefix before the image path, necessary to specify that the path provided is OCI compliant.

Besides, it is interesting to show that Podman copies and extracts the blobs inside its local store (under `$HOME/.local/share/containers/storage` for a rootless container like the one in the previous example).

After learning how to copy images with Skopeo, let\'s see how to inspect remote images without the need to pull them locally.

### Inspecting remote images 

Sometimes we need to verify the configurations, tags, or metadata of an image before pulling and executing it locally. For this purpose, Skopeo offers the useful `skopeo inspect` command to inspect images over supported transports.

The first example shows how to inspect the official NGINX image repository:

``` $ skopeo inspect docker://docker.io/library/nginx ```

The `skopeo inspect` command creates a JSON-formatted output with the following fields:

- `Name`: The name of the image repository.
- `Digest`: The SHA256 calculated digest.
- `RepoTags`: The full list of available image tags in the repository. This list will be empty when inspecting local transports, such as `containers-storage:` or `oci:`, since they will be referred to as a single image.
- `Created`: The creation date of the repository or image.
- `DockerVersion`: The version of Docker used to create the image. This value is empty for images created with Podman, Buildah, or other tools.
- `Labels`: Additional labels applied to the image at build time.
- `Architecture`: The target system architecture for which the image was built. This value is `amd64` for x86-64 systems.
- `Os`: The target operating system the image was built for.
- `Layers`: The list of layers that compose the image, along with their SHA256 digest.
- `Env`: Additional environment variables defined in the image at build time.

The same considerations illustrated previously about authentication and TLS verification apply to the `skopeo inspect` command: it is possible to inspect images on a private registry upon authentication and skip the TLS verification. The next example shows this use case:

``` $ skopeo inspect \ --authfile ${HOME}/.docker/config.json \ --tls-verify false \ registry.example.com/library/test-image ```

Inspecting local images is possible by passing the correct transport. The next example shows how to inspect a local OCI image:

``` $ skopeo inspect oci:/tmp/custom_image ```

The output of this command will have an empty `RepoTags` field.

In addition, it is possible to use the `--no-tags` option to intentionally skip the repository tags, like in the following example:

``` $ skopeo inspect --no-tags docker://docker.io/library/nginx ```

On the other hand, if we need to print only the available repository tags, we can use the `skopeo list-tags` command. The next example prints all the available tags of the official NGINX repository:

``` $ skopeo list-tags docker://docker.io/library/nginx ```

The third use case we are going to analyze is the synchronization of images across registries and local stores.

### Synchronizing registries and local directories 

When working with disconnected environments, a quite common scenario is the need to synchronize repositories from a remote registry locally.

To serve this purpose, Skopeo introduced the `skopeo sync` command, which helps synchronize content between a source and destination, supporting different transport kinds.

We can use this command to synchronize a whole repository, with all the available tags inside it, between a source and a destination. Alternatively, it is possible to synchronize only a specific image tag.

The first example shows how to synchronize the official `busybox` repository from a private registry to the local filesystem. This command pulls all the tags contained in the remote repository to the local destination (the target directory must already exist):

``` $ mkdir /tmp/images $ skopeo sync \ --src docker --dest dir \ registry.example.com/lab/busybox /tmp/images ```

Notice the use of the `--src` and `--dest` options to define the kind of transport. Supported transport types are as follows:

- **Source**: `docker`, `dir`, and `yaml` (covered later in this section)
- **Destination**: `docker` and `dir`

By default, Skopeo syncs the repository content to the destination without the whole image source path. This could represent a limitation when we need to sync repositories with the same name from multiple sources. To solve this limitation, we can add the `--scoped` option and get the full image source path copied in the destination tree.

The second example shows a scoped synchronization of the `busybox` repository:

``` $ skopeo sync \ --src docker --dest dir --scoped \ registry.example.com/lab/busybox /tmp/images ```

The resulting path in the destination directory will contain the registry name and the related namespace, with a new folder named after the image tag.

The next example shows the directory structure of the destination after a successful synchronization:

``` ls -A1 /tmp/images/docker.io/library/ busybox:1 busybox:1.21.0-ubuntu busybox:1.21-ubuntu busybox:1.23 busybox:1.23.2 busybox:1-glibc busybox:1-musl busybox:1-ubuntu busybox:1-uclibc [...omitted output...] ```

If we need to synchronize only a specific image tag, it is possible to specify the tag name in the source argument, as in this third example:

``` $ skopeo sync --src docker --dest dir docker.io/library/busybox:latest /tmp/images ```

We can directly synchronize two registries using Docker, both for the source and destination transport. This is especially useful in disconnected environments where systems are allowed to reach a local registry only. The local registry can mirror repositories from other public or private registries, and the task can be scheduled periodically to keep the mirror updated.

 packt_tip **Important note**

Docker Hub imposes strict pull rate limits on unauthenticated (anonymous) users. If you are following along with these exercises and are not logged into a Docker account, you may quickly hit these limits, causing your image pulls to fail. To avoid *Too Many Requests* errors and ensure a more reliable experience, we recommend using the images hosted on Quay, which currently offers a more generous path for public, unauthenticated pulls.

The next example shows how to synchronize the UBI9 image and all its tags from the public Red Hat repository to a local mirror registry:

``` $ skopeo sync \ --src docker --dest docker \ --dest-tls-verify=false \ registry.access.redhat.com/ubi9 \ mirror-registry.example.com ```

The preceding command will mirror all the UBI9 image tags to the target registry.

Notice the `--dest-tls-verify=false` option to disable TLS certificate checks on the destination.

The `skopeo sync` command is great to mirror repositories and single images between locations, but when it comes to mirroring full registries or a large set of repositories, we should run the command many times, passing different source arguments.

To avoid this limitation, the source transport can be defined as a YAML file to include an exhaustive list of registries, repositories, and images. It is also possible to use regular expressions to capture only selected subsets of image tags.

The following is an example of a custom YAML file that will be passed as a source argument to Skopeo (`Chapter09/example_sync.yaml`):

```
{.programlisting .snippet-code} docker.io: tls-verify: true images: alpine: [] nginx:
- "latest" images-by-tag-regex: httpd: ^2\.4\.[0-9]*-alpine$ quay.io: tls-verify: true images: fedora/fedora:
- latest registry.access.redhat.com: tls-verify: true images: ubi9 :
- "9 .4"
- "9 .5" 
  
```

In the preceding example, different images and repositories are defined, and therefore, the file content deserves a detailed description.

The whole `alpine` repository is pulled from `docker.io`, along with the `nginx:latest` image tag. Also, a regular expression is used to define a pattern of tags for the `httpd` image, in order to pull Alpine-based image version 2.4.z only.

The file also defines a specific tag (`latest`) for the `fedora` image stored under https://quay.io/ and the `9.4` and `9.5` tags for the `ubi9` image stored under the `registry.access.redhat.com` registry.

Once defined, the file is passed as an argument to Skopeo, along with the destination:

``` $ skopeo sync \ --src yaml --dest dir \ --scoped example_sync.yaml /tmp/images ```

All the contents listed in the `example_sync.yaml` file will be copied to the destination directory, following the previously mentioned filtering rules.

The next example shows a larger mirroring use case, applied to the OpenShift release images. The following `openshift_sync.yaml` file defines a regular expression to sync all the images for version 4.17.z of OpenShift built for the x86_64 architecture (`Chapter09/openshift_sync.yaml`):

``` {.programlisting .snippet-code} quay.io: tls-verify: true images-by-tag-regex: openshift-release-dev/ocp-release: ^4\.17 \..*-x86_64$ ```

The **z stream** in a version number (such as `x.y.z`) that typically represents patch releases containing bug fixes and minor improvements that do not introduce new features. Increasing the `z` value indicates a stable, backward-compatible update focused on enhancing the existing version\'s quality.

We can use this file to mirror a whole minor release of OpenShift to an internal registry accessible from disconnected environments and use this mirror to successfully conduct an air-gapped installation of OpenShift Container Platform. The next command example shows this use case:

``` $ skopeo sync \ --src yaml --dest docker \ --dest-tls-verify=false \ --src-authfile pull_secret.json \ openshift_sync.yaml mirror-registry.example.com:5000 ```

It is worth noticing the usage of a pull secret file, passed with the `--src-authfile` option, to authenticate on the Quay public registry and pull images from the `ocp-release` repository.

There is a final Skopeo feature that captures our interest: the remote deletion of images, covered in the next subsection.

### Deleting images 

A registry can be imagined as a specialized object store that implements a set of HTTP APIs to manipulate its content and push/pull objects in the form of image layers and metadata.

The **Docker Registry v2** protocol is a standard API specification that is widely adopted among many registry projects *\[3\]*. This set of API specifications covers all the registry functions that are expected to be exposed to an external client through standard HTTP `GET`, `PUT`, `DELETE`, `POST`, and `PATCH` methods.

This means that we could interact with a registry with any kind of HTTP client capable of managing the requests correctly, for example, the `curl` command.

Any container engine uses, at a lower level, HTTP client libraries to execute the various methods against the registry (for example, for an image pull).

The Docker v2 protocol also supports the remote deletion of images, and any registry that implements this protocol supports the following `DELETE` request for images:

``` DELETE /v2/<name>/manifests/<reference> ```

The following example represents a theoretical `DELETE` command issued with the `curl` command against a local registry:

``` $ curl -v --silent \ -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \ -X DELETE http://127.0.0.1:5000/v2/<name>/manifests/sha256:<image_tag_digest> ```

The preceding example intentionally avoids including the management of authorization tokens for readability.

Podman and Docker, designed to work as registry engines, do not implement a remote `delete` feature among their command interfaces.

Fortunately, Skopeo comes to the rescue with its built-in `skopeo delete` command to manage remote image deletion with a simple and user-friendly syntax.

The following example deletes an image on a hypothetical internal `mirror-registry.example.com:5000` registry:

``` $ skopeo delete \ docker://mirror-registry.example.com:5000/foo:bar ```

The command immediately deletes the image tag references in the remote registry.

When deleting images with Skopeo, it is necessary to enable image deletion in the remote registry, as covered in the next section, *Running a local container registry*.

In this section, we have learned how to use Skopeo to copy, delete, inspect, and sync images or even whole repositories across different transports, including private local registries, gaining control over daily image manipulation operations.

We did not cover the container image\'s signature process here; we will explore it in depth with practical examples in *[Chapter 10](#Chapter_10.xhtml#h1_248){.chapref}*, *Securing* *Containers*.

In the next section, we will learn how to run and configure a local container registry to directly manage image storage in our lab or development environments.


## From Podman build page, my notes

After building, we can *tag* the image with the target registry name. The following example tags the image applying the `v1.0` tag and the `latest` tag:

``` $ podman tag localhost/myhttpd quay.io/<username>/myhttpd:v1.0 ```

After tagging, the image will be ready to be pushed to the remote registry. 


