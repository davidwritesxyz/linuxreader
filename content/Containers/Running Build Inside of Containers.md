Podman and Buildah follow a fork/exec approach that makes them very easy to run inside a container, including rootless container scenarios.

There are many use cases that imply the need for containerized builds. Nowadays, one of the most common adoption scenarios is the application build workflow running on top of a **Kubernetes** cluster.

Kubernetes is basically a container orchestrator that manages the scheduling of containers from a control plane over a set of worker nodes that run a container engine compatible with the **Container Runtime Interface** (**CRI**). Its design allows great flexibility in customizing networking, storage, and runtimes, and leads to the great flourishing of side projects that are now incubating or matured inside the **Cloud Native Computing Foundation** (**CNCF**).

**Vanilla** Kubernetes (which is the basic community release without any customization or add-ons) doesn\'t have a native build feature but offers the proper framework to implement one. Over time, many solutions appeared trying to address this need.

For example, Red Hat **OpenShift** introduced, way back when Kubernetes 1.0 was released, its own build APIs and the **Source-to-Image** toolkit to create container images from source code directly on top of the OpenShift cluster.

Other interesting solutions are Google\'s **kaniko**, which is a build tool to create container images inside a Kubernetes cluster that runs every build step inside user space, and **Cloud Native Buildpacks** (**CNBs**), which offer an approach similar to Source-to-Image with advanced multi-process and single-bill-of-materials management features.

Besides using already-implemented solutions, we can design our own running Buildah inside containers that are orchestrated by Kubernetes. We can also leverage the rootless-ready design to implement secure build workflows.

It is possible to run CI/CD pipelines on top of a Kubernetes cluster and embed containerized builds within a pipeline. One of the most interesting CNCF projects, **Tekton Pipelines**, offers a cloud-native approach to accomplish this goal. Tekton allows running pipelines that are driven by Kubernetes\' custom resources -- special APIs that extend the basic API set.

Tekton Pipelines are made up of many different tasks, and users can either create their own or grab them from **Tekton Hub** (https://hub.tekton.dev/), a free repository where many pre-baked tasks are available to be consumed immediately, including examples from Buildah (https://hub.tekton.dev/tekton/task/buildah).

The preceding examples are useful to understand why containerized builds are important. In this book, we want to focus on the details of running builds within containers, with special attention paid to security-related constraints.

## Running rootless Buildah containers with volume stores 

For the examples in this subsection, the stable upstream `quay.io/buildah/stable` Buildah image will be used. This image already embeds the latest stable Buildah binary.

Let\'s run our first example with a rootless container that builds the contents of the `~/build` directory in the host and stores the output in a local volume named `storevol`:

``` $ podman run --device /dev/fuse \ -v ./http_hello_world:/build:z \ -v storevol:/var/lib/containers quay.io/buildah/stable \ buildah build -t build_test1 /build ```

This example contains some peculiar options that deserve attention:

- The `--device /dev/fuse` option loads the fuse kernel module in the container, which is necessary to run fuse-overlay commands and mount the container filesystem
- The `-v ~/build:/build:z` option bind-mounts the `/build` directory inside the container, assigning proper SELinux labeling with the `:z` suffix
- The `-v storevol:/var/lib/containers` option creates a fresh volume mounted on the default container store, where all the layers are created

When the build is complete, we can run a new container using the same volume and inspect or manipulate the built image:

``` $ podman run --rm -v storevol:/var/lib/containers quay.io/buildah/stable buildah images REPOSITORY TAG IMAGE ID CREATED SIZE localhost/build_test1 latest 3605829966b5 41 seconds ago 33.9 MB registry.access.redhat.com/ubi9/ubi-micro latest e279e18c7ef8 3 days ago 26.4 MB docker.io/library/golang latest 0457bb691895 9 days ago 862 MB ```

We have successfully built an image whose layers have been stored inside the `storevol` volume. To recursively list the content of the store, we can extract the volume mount point with the `podman volume inspect` command:

``` $ ls -alR \ $(podman volume inspect storevol --format '{{.Mountpoint}}') ```

From now on, it is possible to launch a new Buildah container to authenticate with the remote registry, tag the image, and push it. In the following example, Buildah tags the resulting image, authenticates with the remote registry, and then pushes the image:

``` $ podman run --rm -v storevol:/var/lib/containers \ quay.io/buildah/stable \ sh -c 'buildah tag build_test1 \ registry.example.com/build_test1 \ &&buildahlogin-u=<USERNAME> -p=<PASSWORD> \ registry.example.com && \ buildah push registry.example.com/build_test1' ```

When the image is successfully pushed, it is finally safe to remove the volume:

```
# podman volume rm storevol 
```

Despite working perfectly, this approach has some limits that are worth discussing.

The first limit we can notice is that the store volume is not isolated, and thus any other container can access its contents. To overcome this issue, we can use SELinux\'s **Multi-Category Security** (**MCS**) with the `:Z` suffix in order to apply categories to the volume and make it accessible exclusively to the running container.

However, since a second container would run by default with different category labels, we should grab the volume categories and run the second tag/push container with the `--security-opt label=level:s0:<CAT1>,<CAT2>` option.

Alternatively, we can just run `build`, `tag`, and `push` commands in one single container, as shown in the following example:

``` $ podman run --device /dev/fuse \ -v ~/build:/build \ -v secure_storevol:/var/lib/containers:Z \ quay.io/buildah/stable \ sh -c 'buildah build -t test2 /build && \ buildah tag test2 registry.example.com/build_test2 && \ buildah login -u=<USERNAME> \ -p=<PASSWORD> \ registry.example.com && \ buildah push registry.example.com/build_test2' ```

 packt_tip **Important** **note**

In the preceding examples, we used the Buildah login by directly passing the username and password in the command. Needless to say, this is far from being an acceptable security practice.

Instead of passing sensitive data in the command line, we can mount the authentication file that contains a valid session token as a volume inside the container.

The next example mounts a valid `auth.json` file, stored under the `/run/user/<UID>` `tmpfs`, inside the build container, and the `--authfile /auth.json` option is then passed to the `buildah push` command:

``` $ podman run --device /dev/fuse \ -v ~/build:/build \ -v /run/user/<UID>/containers/auth.json:/auth.json:z \ -v secure_storevol:/var/lib/containers:Z \ quay.io/buildah/stable \ sh -c 'buildah build -t test3 /build && \ buildah tag test3 registry.example.com/build_test3 && \ buildah push --authfile /auth.json \ registry.example.com/build_test3' ```

Finally, we have a working example that avoids exposing clear credentials in the commands passed to the container.

To provide a working authentication file, we need to authenticate from the host that will run the containerized build or copy a valid authentication file. To authenticate with Podman, we\'ll use the following command:

``` $ podman login -u <USERNAME> -p <PASSWORD> <REGISTRY> ```

If the authentication process succeeds, the obtained token is stored in the `/run/user/<UID>/containers/auth.json` file, which stores a JSON-encoded object with a structure similar to the following example:

``` {.programlisting .snippet-code} {
 "auths": { "registry.example.com": { "auth": "<base64_encoded_token>" } } }
```

 packt_tip **Security** **alert!**

If the authentication file mounted inside the container has multiple authentication records for different registries, they will be exposed inside the build container. This can lead to potential security issues, since the container will be able to authenticate on those registries using the tokens specified in the file.

The volume-based approach we just described has a small impact on the performance when compared to a native host build but provides better isolation of the build process, a reduced attack surface (thanks to the rootless execution), and standardization of the build environment across different hosts.

Let\'s now inspect how to run containerized builds using bind-mounted stores.

## Running Buildah containers with bind-mounted stores 

In the highest isolation scenario, where DevOps teams follow a zero-trust approach, every build container should have its own isolated store populated at the beginning of the build and destroyed upon completion. Isolation can be easily achieved with SELinux MCS security.

To test this approach, let\'s start by creating a temporary directory that will host the build layers. We also want to generate a random suffix for a name in order to host multiple builds without conflicts:

```
# BUILD_STORE=/var/lib/containers-$(echo $RANDOM | md5sum | head -c 8)
# mkdir $BUILD_STORE 
```

The preceding example and the next builds are executed as root.

We can now run the build and bind-mount the new directory to the `/var/lib/containers` folder inside the container, as well as adding the `:Z` suffix to ensure multi-category security isolation:

```
# podman run --device /dev/fuse \ -v ./build:/build:z \ -v $BUILD_STORE:/var/lib/containers:Z \ -v /run/containers/0/auth.json:/auth.json \ quay.io/buildah/stable \ sh -c 'set -euo pipefail; \ buildah build -t registry.example.com/test4 /build; \ buildah push --authfile /auth.json \ registry.example.com/test4' 
```

The MCS isolation guarantees isolation from other containers. Every build container will have its own custom store, and this implies the need to re-pull the base image layers on every execution, since they are never cached.

Despite being the most secure in terms of isolation, this approach also offers the slowest performance because of the continuous pulls of the base image during the build run.

On the other hand, the less secure approach does not expect any store isolation, and all the build containers mount the default host store under `/var/lib/containers`. This approach provides better performance, since it allows the reuse of cached layers from the host store.

SELinux will not allow a containerized process to access the host store; therefore, we need to relax SELinux security restrictions to run the following example using the `--security-opt label=disable` option.

The following example runs another build using the default host store:

```
# podman run --device /dev/fuse \ -v ./build:/build:z \ -v /var/lib/containers:/var/lib/containers \ --security-opt label=disable \ -v /run/containers/0/auth.json:/auth.json \ quay.io/buildah/stable \ sh -c 'set -euo pipefail; \ buildah build -t registry.example.com/test5 /build; \ buildah push --authfile /auth.json \ registry.example.com/test5' 
```

The approach described in this example is the opposite of the previous one -- better performances but worse security isolation.

A good compromise between the two implies the usage of a secondary, read-only image store to provide access to the cached layers. Buildah supports the usage of multiple image stores, and the `/etc/containers/storage.conf` file *inside the Buildah stable image* already configures the `/var/lib/shared` folder for this purpose.

To prove this, we can inspect the content of the `/etc/containers/storage.conf` file, where the following section is defined:

```
# AdditionalImageStores is used to pass paths to additional Read/Only image stores
# Must be comma separated list. additionalimagestores = [ "/var/lib/shared", ]
```

This way, we can get good isolation and better performance, since cached images from the host will already be available in the read-only store. The read-only store can be pre-populated with the most used images to speed up builds, or can be mounted from a network share.

The following example shows this approach, by bind-mounting the read-only store to the container and executing the build with the advantage of reusing pre-pulled images:

```
# podman run --device /dev/fuse \ -v ./build:/build:z \ -v $BUILD_STORE:/var/lib/containers:Z \ -v /var/lib/containers/storage:/var/lib/shared:ro \ -v /run/containers/0/auth.json:/auth.json:z \ quay.io/buildah/stable \ bash -c 'set -euo pipefail; \ buildah build -t registry.example.com/test6 /build; \ buildah push --authfile /auth.json \ registry.example.com/test6' 
```

The examples shown in this subsection are also inspired by a great technical article written by *Dan Walsh* (one of the leads of the Buildah and Podman projects) on the *Red Hat Developer* blog; refer to the *Further reading* section for the original article link. Let\'s close this section with an example of native Buildah commands.

## Running native Buildah commands inside containers 

We have so far illustrated examples using Dockerfiles/Containerfiles, but nothing prevents us from running containerized native Buildah commands. The following example creates a custom Python image built from a Fedora base image:

```
# BUILD_STORE=/var/lib/containers-$(echo $RANDOM | md5sum | head -c 8)# mkdir $BUILD_STORE

# podman run --device /dev/fuse \ -e REGISTRY=<USER_DEFINED_REGISTRY:PORT> \ --security-opt label=disable \ -v $BUILD_STORE:/var/lib/containers:Z \ -v /var/lib/containers/storage:/var/lib/shared:ro \ -v /run/containers/0:/run/containers/0 \ quay.io/buildah/stable \ sh -c 'set -euo pipefail; \ container=$(buildah from fedora); \ buildah run $container dnf install -y python3 python3; \ buildah commit $container $REGISTRY/python_demo; \ buildah push -authfile \ /run/containers/0/auth.json $REGISTRY/python_demo' 
```

From a performance standpoint, as well as the build process, nothing changes from the previous examples. As already stated, this approach provides more flexibility in the build operations.

If the commands to be passed are too many, a good workaround can be to create a shell script and inject it into the Buildah image using a dedicated volume:

```
# BUILD_STORE=/var/lib/containers-$(echo $RANDOM | md5sum | head -c 8)
# PATH_TO_SCRIPT=/path/to/script
# REGISTRY=<USER_DEFINED_REGISTRY:PORT>
# mkdir $BUILD_STORE
# podman run --device /dev/fuse \ -v $BUILD_STORE:/var/lib/containers:Z \ -v /var/lib/containers/storage:/var/lib/shared:ro \ -v /run/containers/0:/run/containers/0 \ -v $PATH_TO_SCRIPT:/root:z \ quay.io/buildah/stable /root/build.sh 
```

`build.sh` is the name of the shell script file containing all the build custom commands.

In this section, we have learned how to run Buildah in containers covering both volume mounts and bind mounts. We have learned how to run rootless build containers that can be easily integrated into pipelines or Kubernetes clusters to provide an end-to-end application life cycle workflow. This is due to the flexible nature of Buildah, and for the same reason, it is very easy to embed Buildah inside custom builders, as we will see in the next section.
