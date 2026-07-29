
As mentioned before, the multistage build feature is a great approach to produce images with a small footprint and a reduced attack surface. To provide greater flexibility during the build process, the Buildah native commands come to our rescue. As we mentioned earlier in *[Chapter 6](#Chapter_6.xhtml#h1_163){.chapref}*, *Meet Buildah --
Building Containers from Scratch*, Buildah offers a series of commands that replicate the behavior of the Dockerfile instructions, thus offering greater control over the build process when those commands are included in scripts or automations.

The same concept applies when working with multistage builds, where we can also apply extra steps between the stages. For instance, we can mount the build container overlay filesystem and extract the built artifact to release alternate packages, all before building the final runtime image.

The following example builds the same `hello-world` Go application by translating the previous Dockerfile instructions into native Buildah commands, with everything inside a simple shell script:

`Chapter07/http_hello_world/buildah-commands.sh`

```
#!/bin/bash
# Define builder and runtime images BUILDER=docker.io/library/golang RUNTIME=registry.access.redhat.com/ubi9/ubi-micro:latest

# Create builder container 
container1=$(buildah from $BUILDER)

# Copy files from host 
if [ -f go.mod ]; 
then buildah copy $container1 'go.mod' '/go/src/hello-world/' 
else exit 1 
fi 

if [ -f main.go ]; 
then buildah copy $container1 'main.go' '/go/src/hello-world/' 
else exit 1 
fi

# Configure and start build 
buildah config --workingdir /go/src/hello-world $container1 buildah run $container1 go get -d -v ./... buildah run $container1 go build -v ./...

# Create runtime container 
container2=$(buildah from $RUNTIME)

# Copy files from the builder container 
buildah copy --chown=1001:1001 \ 
--from=$container1 $container2 \
 '/go/src/hello-world/hello-world' '/'

# Configure exposed ports 
buildah config --port 8080 $container2

# Configure default CMD 
buildah config --cmd /hello-world $container2

# Configure default user 
buildah config --user=1001 $container2

# Commit final image 
buildah commit $container2 helloworld

# Remove build containers 
buildah rm $container1 $container2 
```

In the preceding example, we highlighted the two working containers\' creation commands and the related `container1` and `container2` variables that store the container ID.

Also, note the `buildah copy` command, where we have defined the source container with the `--from` option, and used the `--chown` option to define user and group owners of the copied resource. This approach proves to be more flexible than the Dockerfile-based workflow, since we can enrich our script with variables, conditionals, and loops.

For instance, we have tested with the `if` condition in the Bash script to check the existence of the `go.mod` and `main.go` files before copying them inside the working container dedicated to the build.

Let\'s now add an extra feature to the script. In the following example, we evolved the previous one by adding semantic versioning for the build and creating a version archive before starting the build of the final runtime image:

The concept of semantic versioning is aimed at providing a clear and standardized way to manage software versioning and dependency management. It is a set of standard rules whose purpose is to define how software release versions are applied, and follows the *X.Y.Z* versioning pattern, where *X* is the major version, *Y* is the minor version, and *Z* is the patch version. For more information, check out the official specifications: https://semver.org/.

`Chapter07/http_hello_world/buildah-commands-release.sh`

```
#!/bin/bash
# Define builder and runtime images BUILDER=docker.io/library/golang RUNTIME=registry.access.redhat.com/ubi9/ubi-micro:latest RELEASE=1.0.0

# Create builder container container1=$(buildah from $BUILDER)

# Copy files from host if [ -f go.mod ]; then buildah copy $container1 'go.mod' '/go/src/hello-world/' else exit 1 fi if [ -f main.go ]; then buildah copy $container1 'main.go' '/go/src/hello-world/' else exit 1 fi

# Configure and start build buildah config --workingdir /go/src/hello-world $container1 buildah run $container1 go get -d -v ./... buildah run $container1 go build -v ./...
# Extract build artifact and create a version archive buildah unshare --mount mnt=$container1 \ sh -c 'cp $mnt/go/src/hello-world/hello-world .' cat > README << EOF Version $RELEASE release notes:
- Implement basic features EOF tar zcf hello-world-${RELEASE}.tar.gz hello-world README rm -f hello-world README

# Create runtime container container2=$(buildah from $RUNTIME)

# Copy files from the builder container buildah copy --chown=1001:1001 \ --from=$container1 $container2 \ '/go/src/hello-world/hello-world' '/'

# Configure exposed ports buildah config --port 8080 $container2

# Configure default CMD buildah config --cmd /hello-world $container2

# Configure default user buildah config --user=1001 $container2

# Commit final image buildah commit $container2 hello world:$RELEASE

# Remove build containers buildah rm $container1 $container2
```

The key changes in the script are again highlighted in bold. First, we added a `RELEASE` variable that tracks the release version of the application. Then, we extracted the build artifact using the `buildah unshare` command, followed by the `--mount` option to pass the container mount point. The `unshare` user namespace was necessary to make the script capable of running rootless.

After extracting the artifact, we created a gzipped archive using the `$RELEASE` variable inside the archive name and removed the temporary files.

Finally, we started the build of the runtime image and committed using the `$RELEASE` variable again as the image tag.

In this section, we have learned how to run multistage builds with Buildah using both Dockerfiles/Containerfiles and native commands. In the next section, we will learn how to isolate Buildah builds inside a container.


# Integrating Buildah with custom builders 

As we saw in the previous section of this chapter, Buildah is a key component of Podman\'s container ecosystem. Buildah is a dynamic and flexible tool that can be adapted to different scenarios to build brand-new containers. It has several options and configurations available, but our exploration is not yet finished.

Podman and all the projects developed around it have been built with extensibility in mind, making every programmable interface available to be reused from the outside world.

Podman, for example, inherits Buildah capabilities for building brand-new containers through the `podman build` command; with the same principle, we can embed Buildah interfaces and its engine in our custom builder.

Let\'s see how to build a custom builder in the Go language; we will see that the process is pretty straightforward, because Podman, Buildah, and many other projects in this ecosystem are actually written in the Go language.

## Including Buildah in our Go build tool 

As a first step, we need to prepare our development environment, downloading and installing all the required tools and libraries for creating our custom build tool.

 packt_tip **Important** **note**

Please be aware that this is an unsupported scenario and the stability in terms of deprecations and changes of the Buildah Go APIs is not guaranteed. For supported and stable use cases, refer to the Podman REST APIs.

In *[Chapter 3](#Chapter_3.xhtml#h1_83){.chapref}*, *Running* *the First Container*, we saw various Podman installation methods. In the following section, we will use a similar procedure while going through the preliminary steps for building a Buildah project from scratch, downloading its source file to include in our custom builder.

First of all, let\'s ensure we have all the needed packages installed on our development host system:

```
# dnf install -y golang git go-md2man btrfs-progs-devel gpgme-devel device-mapper-devel Fedora 40
- x86_64 253 kB/s | 28 kB 00:00 Fedora 40 openh264 (From Cisco)
- x86_64 9.5 kB/s | 989 B 00:00 Fedora 40
- x86_64
- Updates 141 kB/s | 25 kB 00:00 Fedora 40
- x86_64
- Updates 3.9 MB/s | 6.2 MB 00:01 Dependencies resolved. ============================================================================================================================================================================================== Package Architecture Version Repository Size ============================================================================================================================================================================================== Installing: btrfs-progs-devel x86_64 6.11-1.fc40 updates 49 k device-mapper-devel x86_64 1.02.199-1.fc40 updates 41 k git x86_64 2.47.0-1.fc40 updates 52 k golang x86_64 1.22.7-1.fc40 updates 666 k golang-github-cpuguy83-md2man x86_64 2.0.3-3.fc40 fedora 748 k gpgme-devel x86_64 1.23.2-3.fc40 fedora 167 k U
 [... omitted output] 
```

After installing the Go language core libraries and some other development tools, we are ready to create the directory structure for our project and initialize it:

``` $ mkdir ~/custombuilder $ cd ~/custombuilder [custombuilder]$ export GOPATH=`pwd` ```

As shown in the previous example, we followed these steps:

1. Created the project root directory 2. Defined the Go language root path that we are going to use

We are now ready to create our Go module that will create our customized container image with a few easy steps.

To speed up the example and avoid any writing errors, we can download the Go language code that we are going to use for this test from the official GitHub repository of this book:

1. Go to https://github.com/PacktPublishing/Podman-for-DevOps or run the following command:

 ``` {.programlisting .snippet-con-one} $ git clone https://github.com/PacktPublishing/Podman-for-DevOps ```

2. After that, copy the files provided in the `Chapter07/*` directory into the newly created `~/custombuilder/` directory.

 You should have the following files in your directory at this point:

 ``` {.programlisting .snippet-con-one} $ cd ~/custombuilder/src/builder $ ls -la total 148 drwxrwxr-x. 1 alex alex 74 9 nov 15.22 . drwxrwxr-x. 1 alex alex 14 9 nov 14.10 .. -rw-rw-r--. 1 alex alex 1466 9 nov 14.10 custombuilder.go -rw-rw-r--. 1 alex alex 161 9 nov 15.22 go.mod -rw-rw-r--. 1 alex alex 135471 9 nov 15.22 go.sum -rw-rw-r--. 1 alex alex 337 9 nov 14.17 script.js ```

 At this point, we can run the following command to let the Go tools acquire all the needed dependencies to ready the module for execution:

 ``` {.programlisting .snippet-con-one} $ go mod tidy -v go: finding module for package github.com/containers/storage/pkg/unshare go: finding module for package github.com/containers/image/v5/storage[...omitted output...] ```

 The tool analyzed the provided `custombuilder.go` file, and it found all the required libraries, populating the `go.mod` file.

 ::: packt_tip-one **Important** **note**

 Please be aware that the previous command will verify whether a module is available, and if it is not, the tool will start downloading it from the internet. So, be patient during this step! :::

 We can check that the previous commands downloaded all the required packages by inspecting the directory structure we created earlier:

 ``` {.programlisting .snippet-con-one} $ cd ~/custombuilder [custombuilder]$ ls pkg src [custombuilder]$ ls -la pkg/ total 0 drwxrwxr-x. 1 alex alex 28 9 nov 18.27 . drwxrwxr-x. 1 alex alex 12 9 nov 18.18 .. drwxrwxr-x. 1 alex alex 20 9 nov 18.27 linux_amd64 drwxrwxr-x. 1 alex alex 196 9 nov 18.27 mod [custombuilder]$ ls -la pkg/mod/ total 0 drwxrwxr-x. 1 alex alex 196 9 nov 18.27 . drwxrwxr-x. 1 alex alex 28 9 nov 18.27 .. drwxrwxr-x. 1 alex alex 22 9 nov 18.18 cache drwxrwxr-x. 1 alex alex 918 9 nov 18.27 github.com drwxrwxr-x. 1 alex alex 24 9 nov 18.27 go.etcd.io drwxrwxr-x. 1 alex alex 2 9 nov 18.27 golang.org [... omitted output] [custombuilder]$ ls -la pkg/mod/github.com/ [... omitted output] drwxrwxr-x. 1 alex alex 98 9 nov 18.27 containerd drwxrwxr-x. 1 alex alex 20 9 nov 18.27 containernetworking drwxrwxr-x. 1 alex alex 184 9 nov 18.27 containers drwxrwxr-x. 1 alex alex 110 9 nov 18.27 coreos [... omitted output] ```

We are now ready to run our custom builder module, but before moving forward, let\'s take a look at the key elements contained in the Go source file.

 packt_tip **Important** **note**

Please consider that on Fedora 40, as well as for other Linux distributions, you may also need additional development packages from your distribution\'s repositories. For Fedora, for example, in order to successfully run the Go program, you may also need to install the `btrfs-progs-devel` and `libgpgme-devel` packages. Please refer to Podman\'s documentation for more info:

https://podman.io/docs/installation#build-and-run-dependencies

If we start looking at the `custombuilder.go` file, just after defining the package and the libraries to use, we define the main function of our module.

In the main function, at the beginning of the definition, we inserted a fundamental code block:

``` {.programlisting .snippet-code} if buildah.InitReexec() { return } unshare.MaybeReexecUsingUserNamespace(false) ```

This piece of code enables the usage of **rootless** mode by leveraging the Go `unshare` package, available through `github.com/containers/storage/pkg/unshare`.

To leverage the build features of Buildah, we have to instantiate `buildah.Builder`. This object has all the methods to define the build steps, configure the build, and finally run it.

To create `Builder`, we need an object called `storage.Store` from the `github.com/containers/storage` package. This element is responsible for storing the intermediate and result container images. Let\'s see the code block we are discussing:

``` {.programlisting .snippet-code} buildStoreOptions, err := storage.DefaultStoreOptions( ) buildStore, err := storage.GetStore(buildStoreOptions) ```

As you can see from the previous example, we are getting the default options and passing them to the `storage` module to request a `Store` object.

Another element we need for creating `Builder` is the `BuilderOptions` object. This element contains all the default and custom options we might assign to Buildah\'s `Builder`. Let\'s see how to define it:

```

builderOpts := buildah.BuilderOptions{ FromImage: "node:23-alpine", // Starting image Isolation: define.IsolationChroot, // Isolation environment CommonBuildOpts: &define.CommonBuildOptions{}, ConfigureNetwork: define.NetworkDefault, SystemContext: &types.SystemContext {}, }
```

In the previous code block, we defined a `BuilderOptions` object that contains the following:

- An initial image that we are going to use to build our target container image:
- In this case, we chose the Node.js image based on the Alpine Linux distribution. This is because, in our example, we are simulating the build process of a Node.js application.
- Isolation mode to adopt once the build starts. In this case, we are going to use chroot isolation, which fits a lot of build scenarios well -- less isolation but fewer requirements.
- Some default options for the build, network, and system contexts:
- `SystemContext` objects define the information contained in configuration files as parameters

Now that we have all the necessary data for instantiating `Builder`, let\'s do it:

```
builder, err := buildah.NewBuilder(context.TODO(), buildStore, builderOpts) 
```

As you can see, we are calling the `NewBuilder` function, with all the required options that we created in code earlier in this section, to get `Builder` ready to create our custom container image.

We are now ready to instruct `Builder` with the required options to create the custom image. Let\'s first add to the container image the **JavaScript** file containing our application, for which we are creating this container image:

``` {.programlisting .snippet-code} err = builder.Add("/home/node/", false, buildah.AddAndCopyOptions{}, "script.js") ```

We are assuming that the JavaScript main file is stored next to the Go module that we are writing and using in this example, and we are copying this file into the `/home/node` directory, which is the default path where the base container image expects to find this kind of data.

The JavaScript program that we are going to copy into the container image and use for this test is really simple -- let\'s inspect it:

``` {.programlisting .snippet-code} var http = require("http"); http.createServer(function(request, response) { response.writeHead(200, {"Content-Type": "text/plain"}); response.write("Hello Podman and Buildah friends. This page is provided to you through a container running Node.js version: "); response.write(process.version); response.end(); }).listen(8080); ```

Without going deep into the JavaScript language syntax and its concepts, we can note, by looking at the JavaScript file, that we are using the HTTP library for listening on port `8080` for incoming requests, responding to these requests with a default welcome message: `Hello Podman and Buildah friends. This page is provided to you through a container running Node.js`. We also append the Node.js version to the response string.

Please consider that **JavaScript**, also known as **JS**, is a high-level programming language that is compiled just in time. As we stated earlier, we are not going to go deep into the definition of the JavaScript language or its most famous runtime environment, Node.js.

After that, we configure the default command to run for our custom container image:

```
builder.SetCmd([]string{"node", "/home/node/script.js"}) 
```

We just set the command to execute the Node.js execution runtime, referring to the JavaScript program that we just added to the container image.

To commit the changes we made, we need to get the image reference that we are working on. At the same time, we will also define the name of the container image that `Builder` will create:

``` {.programlisting .snippet-code} imageRef, err := is.Transport.ParseStoreReference(buildStore, "podmanbook/nodejs-welcome") ```

Now, we are ready to commit the changes and call the `commit` function of `Builder`:

``` {.programlisting .snippet-code} imageId, _, _, err := builder.Commit(context.TODO(), imageRef, define.CommitOptions{}) fmt.Printf("Image built! %s\n", imageId) ```

As we can see, we just requested `Builder` to commit the changes, passing the image reference we obtained earlier, and then we finally print it as a reference.

We are now ready to run our program! Let\'s execute it:

``` [builder]$ go run custombuilder.go Image built! e60fa98051522a51f4585e46829ad6a18df704dde774634dbc010baae4404849 ```

We can now test the custom container image we just built:

``` [builder]$ podman run -dt -p 8080:8080/tcp podmanbook/nodejs-welcome:latest 747805c1b59558a70c4a2f1a1d258913cae5ffc08cc026c74ad3ac21aab18974 [builder]$ curl localhost:8080 Hello Podman and Buildah friends. This page is provided to you through a container running Node.js version: v23.1.0 ```

As we can see in the previous code block, we are running the container image we just created with the following options:

- `-d`: Detached mode, which runs the container in the background
- `-t`: Allocates a new pseudo-TTY
- `-p`: Publishes the container port to the host system
- `podmanbook/nodejs-welcome:latest`: The name of our custom container image

Finally, we use the `curl` command-line tool for requesting and printing the HTTP response provided by our JavaScript program, which is containerized in the custom container image that we created!

The example described in this section is just a simple overview of all the great features that the Buildah Go module can enable for our custom image builders. To learn more about the various functions, variables, and code documentation, you can refer to the docs at https://pkg.go.dev/github.com/containers/buildah.

As we saw in this section, Buildah is a really flexible tool, and with its libraries, it can support custom builders in many different scenarios.

If we try to search on the internet, we can find many examples of Buildah supporting the creation of custom container images. Let\'s see some of them.

## Running Quarkus-native executables in containers 

**Quarkus** is defined as the Kubernetes-native Java stack leveraging the OpenJDK (the open Java development kit) project and the GraalVM project. GraalVM is a Java virtual machine that has many special features, such as the compilation of Java applications for fast startup and a low memory footprint.

We will not go into the details of Quarkus, GraalVM, and any other companion projects. The example that we will deep-dive into is only for your reference. We encourage you to learn more about these projects by going through their web pages and reading the related documentation.

If we take a look at the Quarkus documentation web page, we can easily find that, after a long tutorial in which we can learn how to build a Quarkus-native executable, we can then pack and execute this executable in a container image.

The steps provided in the Quarkus documentation leverage a Maven wrapper with a special option. Maven was created as a Java build automation tool, but then it was also extended to other programming languages. If we take a quick look at this command, we will notice that `podman` is mentioned:

``` $ ./mvnw package -Pnative -Dquarkus.native.container-build=true -Dquarkus.native.container-runtime=podman ```

This means that the Maven wrapper program will invoke a Podman build to create a container image with the preconfigured environment shipped by the Quarkus project and the binary application that we are developing.

Podman is mentioned because, as we saw in *[Chapter 6](#Chapter_6.xhtml#h1_163){.chapref}*, *Meet Buildah -- Building Containers from Scratch*, it borrows Buildah\'s build logic by vendoring its libraries.

To explore this example further, take a look at https://quarkus.io/guides/building-native-image.

# Further reading

- A list of CNCF projects: https://landscape.cncf.io/
- Best practices for running Buildah in a container: https://developers.redhat.com/blog/2019/08/14/best-practices-for-running-buildah-in-a-container
- The Buildah Go module documentation: https://pkg.go.dev/github.com/containers/buildah
- Quarkus-native executables: https://quarkus.io/guides/building-native-image
