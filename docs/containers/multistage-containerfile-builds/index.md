# 

# Integrating with Existing Application Build Processes 

- Multistage container builds
- Running Buildah inside a container
- Integrating Buildah with custom builders

# Multistage container builds 

size of the images
- Final image size is the result of the total number of layers and the number of changed files inside them.
- Minimal images with a small size have the great advantage of being able to be pulled faster from registries.
- Large images will eat a lot of disk space in the host's local store.

To keep images compact in size:
- Build images from scratch.
- Cleaning up package manager caches.
- Reduce the amount of `RUN`, `COPY`, and `ADD` instructions to the minimum necessary. 

To build a containerized Go application:
	- Start from a base image that includes Go runtimes
	- Copy the source code.
	- Compile to produce the final binary with a series of intermediate steps
		- Downloading all the necessary Go packages inside the image cache. 
	- At the end of the build, clean up all the source code and the downloaded dependencies and put the final binary (which is statically linked in Go) in a working directory. 
- Everything will work, but the final image will still include the Go runtimes included in the base image, which are no longer necessary at the end of the compilation process.

**binary builds** 
- A way to inject the final artifact compiled externally inside the built image. 
- Solves the image size problem but removes the advantage of a standardized environment for builds provided by runtime/compiler images.

- A better approach is to share volumes between containers and have the final container image grab the compiled artifacts from a first build image.

**multistage builds**
- Allow users to create builds with multiple stages using different `FROM` instructions and have subsequent images grab contents from the previous ones.

## Multistage builds with Dockerfiles 

The first approach to multistage builds 
- Creating multiple stages in a single Dockerfile/Containerfile, with each block beginning with a `FROM` instruction.
- Build stages can copy files and folders from previous ones using the `--from` option to specify the source stage.

Create a minimal multistage build for the Go application, with the first stage acting as a pure build context and the second stage copying the final artifact inside a minimal image.

This example is defined in the following file:

```
Stage 1
# Builder image 
FROM docker.io/library/golang

# Copy files for build 
COPY go.mod /go/src/hello-world/ 
COPY main.go /go/src/hello-world/

# Set the working directory 
WORKDIR /go/src/hello-world

# Download dependencies 
RUN go get -d -v ./...

# Install the package 
RUN go build -v

Stage 2
# Runtime image 
FROM registry.access.redhat.com/ubi9/ubi-micro:latest 
COPY --from=0 /go/src/hello-world/hello-world / 
EXPOSE 8080

CMD ["/hello-world"] 
```

Stage 1
- Copies the source `main.go` file and the `go.mod` file to manage the Go module dependencies.
- After downloading the dependency packages (`go get -d -v ./...`), the final application is built (`go build -v ./...`).

Stage 2
- Grabs the final artifact (`/go/src/hello-world/hello-world`) 
- Copies it under the new image root. 
- To specify that the source file should be copied from the first stage, the `--from=0` syntax is used.

The Go application listed as follows is a basic web server that listens on port `8080/tcp` and prints a crafted HTML page with the **\"Hello World!\"** message when it receives a `GET /` request:

`Chapter07/http_hello_world/main.go`

``` {.programlisting .snippet-code} package main

import ( "log" "net/http" )

func handler(w http.ResponseWriter, r *http.Request) { log.Printf("%s %s %s\n", r.RemoteAddr, r.Method, r.URL) w.Header().Set("Content-Type", "text/html") w.Write([]byte("<html>\n<body>\n")) w.Write([]byte("<p>Hello World!</p>\n")) w.Write([]byte("</body>\n</html>\n")) }

func main() { http.HandleFunc("/", handler) log.Println("Starting http server") log.Fatal(http.ListenAndServe(":8080", nil)) }
```

The application can be built using either Podman or Buildah. In this example, we chose to build the application with Buildah:

``` $ cd http_hello_world $ buildah build -t hello-world . ```

Finally, we can check the resulting image size:
``` $ buildah images --format '{{.Name}} {{.Size}}' \ localhost/hello-world localhost/hello-world 36.1 MB ```

The final image has a size of only 36 MB!

We can improve our Dockerfile by adding custom names to the base images using the `AS` keyword. The following example is a rework of the previous Dockerfile following this approach, with the key elements highlighted in bold:

`Chapter07/http_hello_world/Dockerfile-builder`

```
# Builder image 
FROM docker.io/library/golang AS builder

# Copy files for build 
COPY go.mod /go/src/hello-world/ 
COPY main.go /go/src/hello-world/

# Set the working directory 
WORKDIR /go/src/hello-world

# Download dependencies 
RUN go get -d -v ./...

# Install the package 
RUN go build -v ./...

# Runtime image 
FROM registry.access.redhat.com/ubi9/ubi-micro:latest AS srv 
COPY --from=builder /go/src/hello-world/hello-world / 
EXPOSE 8080

CMD ["/hello-world"] 
```

In the preceding example, the name of the builder image is set as `builder`, while the final image is named `srv`. Interestingly, the `COPY` instruction can now specify the builder as using the custom name with the `--from=builder` option.

We can again build the container image with the following command:

``` $ cd http_hello_world $ buildah build -f ./Dockerfile-builder -t hello-world-v2 . ```

Dockerfile/Containerfile builds are the most common approach but still lack some flexibility when it comes to implementing a custom build workflow. For those special use cases, Buildah native commands come to our rescue.