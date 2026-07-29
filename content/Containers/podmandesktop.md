
Managing Your Container, Kubernetes, and AI Workloads from a Graphical Interface


- Accelerate day-to-day tasks such as inspecting logs, managing multiple environments, and exploring new technologies.
- Comprehensive control center for all your container, Kubernetes, and even **artificial** **intelligence** (**AI**) development needs, right on your local machine.

In this chapter, we\'re going to cover the following main topics:

-   Setting up the prerequisites for the host operating system
-   Easily building, running, stopping, and managing containers with a
    user-friendly interface
-   Connecting to and interacting directly with Kubernetes clusters
-   Provisioning local inference servers to run AI models and obtaining
    endpoints for application integration
-   Integrating an AI model running on Podman for building your own
    chatbot use case

# Technical requirements {#Chapter_15.xhtml#h1_350 .heading-1}

To complete this chapter, you will need a machine with a working Podman
installation.[]{.sentence-end} As we mentioned in *[Chapter
3](#Chapter_3.xhtml#h1_83){.chapref}*, *Running* *the* *First
Container*, all the examples in this book were executed on a Fedora 40
system or later, but can be reproduced on your choice of **operating
system** (**OS**).

Having a good understanding of the topics that were covered in *[Chapter
4](#Chapter_4.xhtml#h1_114){.chapref}*, *Managing Running Containers*,
*[Chapter 5](#Chapter_5.xhtml#h1_133){.chapref}*, *Implementing Storage
for the Container\'s Data*, and *[Chapter
9](#Chapter_9.xhtml#h1_219){.chapref}*, *Pushing Images to a Container
Registry*, will help you grasp the topics we\'ll cover in this chapter.

You should also have a good understanding of system administration and
Kubernetes container orchestration, and a general understanding of
generative AI.

# Setting up the prerequisites for the host operating system {#Chapter_15.xhtml#h1_351 .heading-1}

Before []{#Chapter_15.xhtml#idx_e77c0b62 .index-entry
index-entry="host OS:prerequisites"}we can dive into managing our
container and AI workloads, we first need to install the primary tool
for this chapter: **Podman** **Desktop**.[]{.sentence-end} Podman
Desktop []{#Chapter_15.xhtml#idx_24eb8b44 .index-entry
index-entry="Podman Desktop"}is an open source graphical tool that
enables you to seamlessly work with containers and Kubernetes,
regardless of your local environment.[]{.sentence-end} It provides a
unified interface to manage containers, build images, interact with
container registries, and connect []{#Chapter_15.xhtml#idx_c733b6bf
.index-entry index-entry="host OS:prerequisites"}to Kubernetes clusters.

In the following subsections, we will walk through the installation
process for Linux, macOS, and Windows.

## Installing on Linux {#Chapter_15.xhtml#h2_352 .heading-2}

For Linux users, Podman Desktop can be installed in a couple of
ways.[]{.sentence-end} We will cover the universal method using Flatpak,
which works across most distributions, and the native package manager
method for Fedora users.

### Installing with Flatpak {#Chapter_15.xhtml#h3_353 .heading-3}

Flatpak[]{#Chapter_15.xhtml#idx_b34ffd90 .index-entry
index-entry="Flatpak"} is a universal packaging system for Linux that
allows applications to run in a sandboxed environment, isolated from the
rest of the system.[]{.sentence-end} This is the recommended method for
most Linux distributions:

1.  First, ensure that []{#Chapter_15.xhtml#idx_3771edb9 .index-entry
    index-entry="Flatpak:Podman Desktop, installing on Linux"}you have
    Flatpak installed and []{#Chapter_15.xhtml#idx_51de71bd .index-entry
    index-entry="Linux:Podman Desktop, installing on"}the Flathub
    repository configured []{#Chapter_15.xhtml#idx_e2026776 .index-entry
    index-entry="Podman Desktop:installing, on Linux"}on your
    system.[]{.sentence-end} If not, please refer to the official
    []{#Chapter_15.xhtml#idx_2dab7764 .index-entry
    index-entry="Flatpak:reference link, for setup guide"}Flatpak setup
    guide at
    [[https://flatpak.org/setup/]{.url}](https://flatpak.org/setup/){style="text-decoration: none;"}.[]{.sentence-end}
    Once Flatpak is ready, you can install Podman Desktop with a single
    command:

    ``` {.programlisting .snippet-con-one}
    # flatpak install flathub io.podman_desktop.PodmanDesktop
    ```

2.  Flatpak will resolve the required runtimes and dependencies, ask for
    confirmation, and proceed with the installation.

3.  After the installation is complete, you can launch Podman Desktop
    from your distribution\'s application menu or by running the
    following command from the terminal:

    ``` {.programlisting .snippet-con-one}
    # flatpak run io.podman_desktop.PodmanDesktop
    ```

4.  Upon the []{#Chapter_15.xhtml#idx_9a516773 .index-entry
    index-entry="Flatpak:Podman Desktop, installing on Linux"}first
    launch, Podman Desktop[]{#Chapter_15.xhtml#idx_5b51700b .index-entry
    index-entry="Linux:Podman Desktop, installing on"} will greet you
    with a welcome screen[]{#Chapter_15.xhtml#idx_d256beaf .index-entry
    index-entry="Podman Desktop:installing, on Linux"} and check for any
    required dependencies, such as a working Podman engine
    installation.[]{.sentence-end}

    <figure class="mediaobject-one">
    <img src="images/B31467_15_1.png"
    style="width:460.79999999999995px; height:239.53983471074378px;"
    alt="Image 1" />
    </figure>

    Figure 15.1: Podman Desktop welcome screen on a Linux system after
    Flatpak installation

## Installing on macOS {#Chapter_15.xhtml#h2_354 .heading-2}

On macOS, Podman[]{#Chapter_15.xhtml#idx_49d47785 .index-entry
index-entry="Podman Desktop:installing, on macOS"} Desktop can be
installed using either the official disk
[]{#Chapter_15.xhtml#idx_c287a75e .index-entry
index-entry="macOS:Podman Desktop, installing on"}image
(`.`{.inlineCode}`dmg`{.inlineCode}) file or via the Homebrew package
manager:

-   **Using the** `.dmg `{.inlineCode}**installer**:
    1.  Navigate to the official Podman Desktop website at
        [[https://podman-desktop.io/]{.url}](https://podman-desktop.io/){style="text-decoration: none;"}.
    2.  Download the installer for macOS (it will be a
        `.`{.inlineCode}`dmg`{.inlineCode} file).
    3.  Open the downloaded `.`{.inlineCode}`dmg`{.inlineCode}
        file.[]{.sentence-end} A window will appear.
    4.  Drag the Podman Desktop icon into your
        `Applications`{.inlineCode} folder.

-   **Using Homebrew**:

    If you have Homebrew installed, you can simply run the following
    command in your terminal:

    ``` {.programlisting .snippet-con-one}
    # brew install --cask podman-desktop
    ```

    After that, you[]{#Chapter_15.xhtml#idx_1354c6fd .index-entry
    index-entry="Podman Desktop:installing, on macOS"} should be able to
    start Podman Desktop and []{#Chapter_15.xhtml#idx_0b2afd45
    .index-entry index-entry="macOS:Podman Desktop, installing on"}see
    something similar to the following:

    <figure class="mediaobject-one">
    <img src="images/B31467_15_2.png"
    style="width:460.79999999999995px; height:235.7033248081841px;"
    alt="Image 2" />
    </figure>

    Figure 15.2: Podman Desktop welcome screen on macOS after
    installation

After installation, you can launch Podman Desktop from your
`Applications`{.inlineCode} folder.[]{.sentence-end} On its first run,
it will guide you through the process of setting up the Podman machine,
which is a lightweight virtual machine required for running Podman on
macOS.

## Installing on Windows {#Chapter_15.xhtml#h2_355 .heading-2}

For Windows []{#Chapter_15.xhtml#idx_3e8889a3 .index-entry
index-entry="Windows:Podman Desktop, installing on"}users, Podman
[]{#Chapter_15.xhtml#idx_f4e6b99b .index-entry
index-entry="Podman Desktop:installing, on Windows"}Desktop leverages
**Windows Subsystem for Linux** (**WSL**) 2 to run the Podman
engine.[]{.sentence-end} The installer simplifies the
entire[]{#Chapter_15.xhtml#idx_e08aed91 .index-entry
index-entry="Windows Subsystem for Linux (WSL) 2"} setup process:

1.  Navigate to the official Podman Desktop website at
    [[https://podman-desktop.io/]{.url}](https://podman-desktop.io/){style="text-decoration: none;"}.
2.  Download the installer for Windows (it will be an
    `.exe`{.inlineCode} file).
3.  Run the downloaded installer.[]{.sentence-end} It will guide you
    through the setup wizard.
4.  Follow the on-screen instructions.[]{.sentence-end} If you do not
    have WSL 2 or a working Podman installation, the setup wizard will
    offer to configure them for you.

Once the installation is complete, Podman Desktop will be available in
your **Start** menu:

<figure class="mediaobject">
<img src="images/B31467_15_3.png"
style="width:528.0px; height:270.33250620347394px;" alt="Image 3" />
</figure>

Figure 15.3: Podman Desktop welcome screen on a Windows OS after
installation

On the first[]{#Chapter_15.xhtml#idx_83d2c5d7 .index-entry
index-entry="Windows:Podman Desktop, installing on"} launch, it will
finalize the WSL 2 integration and ensure
[]{#Chapter_15.xhtml#idx_8b75725d .index-entry
index-entry="Podman Desktop:installing, on Windows"}that the Podman
machine is running correctly, providing a seamless container management
experience on Windows.

 note
**Please** **note**

The installation instructions could change slightly depending on your OS
release.[]{.sentence-end} For additional information, please refer to
the related Podman Desktop website.


With Podman Desktop now installed on our OS, we are ready to explore its
user-friendly interface for managing containers in the next section.

# Easily building, running, stopping, and managing containers with a user-friendly interface {#Chapter_15.xhtml#h1_356 .heading-1}

One of the primary goals of Podman Desktop is to simplify the day-to-day
workflow of developers and operators.[]{.sentence-end} It provides a
visual dashboard to see what\'s running at a glance and offers intuitive
controls for common operations that would otherwise require remembering
various command-line flags.

When you first launch the application, you are presented with the main
dashboard.[]{.sentence-end} The left-hand sidebar is your main
navigation tool, providing access to different resources such as
**Containers**, **Pods**, **Images**, **Volumes**, and
**Settings**.[]{.sentence-end} We will use these sections to manage our
containerized applications.

<figure class="mediaobject">
<img src="images/B31467_15_4.png"
style="width:528.0px; height:274.4727272727273px;" alt="Image 4" />
</figure>

Figure 15.4: The main Podman Desktop dashboard, highlighting the sidebar
and an empty list of containers

## Running a container from an image {#Chapter_15.xhtml#h2_357 .heading-2}

Before we can run []{#Chapter_15.xhtml#idx_45b20d9c .index-entry
index-entry="container:running, from image"}a container, we need an
image.[]{.sentence-end} Let\'s start by pulling a
standard[]{#Chapter_15.xhtml#idx_285cea83 .index-entry
index-entry="image:container, running from"} NGINX web server image from
a public registry and then launch a container from it:

1.  Navigate to the **Images** section using the left
    sidebar.[]{.sentence-end} This screen lists all the container images
    available on your local machine.

2.  Click the **Pull an image** button.[]{.sentence-end} A dialog will
    appear.

3.  In the dialog, enter the name of the image you want to
    pull.[]{.sentence-end} For this example, we will use
    `docker.io/library/nginx`{.inlineCode}.[]{.sentence-end} As you can
    see, we are not setting any tag, so we are pulling only the
    `latest`{.inlineCode} tag.

4.  Click the **Pull image** button to start the
    download.[]{.sentence-end} You will see a progress indicator as
    Podman Desktop downloads the image layers.[]{.sentence-end} Once
    complete, the `nginx`{.inlineCode} image will appear in your list of
    local images.[]{.sentence-end}

    <figure class="mediaobject-one">
    <img src="images/B31467_15_5.png"
    style="width:460.79999999999995px; height:239.53983471074378px;"
    alt="Image 5" />
    </figure>

    Figure 15.5: The Images screen showing the newly pulled NGINX image
    with its details

Now that we have the[]{#Chapter_15.xhtml#idx_853ddd6a .index-entry
index-entry="container:running, from image"} image, we can create and
run a container[]{#Chapter_15.xhtml#idx_f90f0e30 .index-entry
index-entry="image:container, running from"} from it:

1.  In the **Images** list, locate the `nginx`{.inlineCode} image and
    click the **Run** icon (a *play* symbol) in the **Actions** column.

2.  This will bring up the **Create a container** configuration
    screen.[]{.sentence-end} Here, you can define all the parameters for
    your new container:

    -   **Container** **name**: Give your container a memorable name,
        such as `my-web-server`{.inlineCode}.
    -   **Port** **mapping**: This is crucial for accessing the web
        server from your host machine.[]{.sentence-end} In the **Ports**
        section, map a local host port to the container\'s exposed
        port.[]{.sentence-end} NGINX listens on port `80`{.inlineCode}
        inside the container, so let\'s map port `8080`{.inlineCode} on
        our host to port `80`{.inlineCode} in the container by entering
        `8080`{.inlineCode} in the **Host Port** field and
        `80`{.inlineCode} in the **Container Port**
        field.[]{.sentence-end} In case Podman Desktop recognizes the
        existing exposed ports, you will need to enter just the target
        host port.

    <figure class="mediaobject-one">
    <img src="images/B31467_15_6.png"
    style="width:460.79999999999995px; height:239.53983471074378px;"
    alt="Image 6" />
    </figure>

    Figure 15.6: The Create a container dialog, with the container name
    and port mapping fields filled out

3.  Click the **Start Container** button.[]{.sentence-end} Podman
    Desktop will create and start the container.[]{.sentence-end} It
    []{#Chapter_15.xhtml#idx_ce668195 .index-entry
    index-entry="container:running, from image"}will then automatically
    navigate you []{#Chapter_15.xhtml#idx_466e854d .index-entry
    index-entry="image:container, running from"}to the **Containers**
    list, where you will see your `my-web-server`{.inlineCode} container
    running, indicated by a green icon and a **Running** status label.

To verify that it\'s working, open a web browser and navigate to
[[http://localhost:8080]{.url}](https://http://localhost:8080){style="text-decoration: none;"}.[]{.sentence-end}
You should see the default **Welcome to nginx!** page.

## Managing the container lifecycle {#Chapter_15.xhtml#h2_358 .heading-2}

The **Containers** list is your[]{#Chapter_15.xhtml#idx_88fad17a
.index-entry index-entry="container lifecycle:managing"} central hub for
managing all running and stopped containers.[]{.sentence-end} For each
container, you have a set of quick actions to control its state:

-   **Stop/Start**: The square stop icon will stop a running container,
    changing its status to **Exited**.[]{.sentence-end} The play icon
    will start a stopped container.
-   **Delete**: The trash can icon allows you to permanently remove a
    container.[]{.sentence-end} Note that a container must be stopped
    before it can be deleted.

For more detailed interaction, you can click on the container\'s name to
open its **Details** view.[]{.sentence-end} This view is organized into
several tabs:

-   **Logs**: This tab provides a real-time stream of the container\'s
    standard output, which is invaluable for debugging.[]{.sentence-end}
    You can see NGINX\'s access logs here as you refresh your browser
    page.

-   **Inspect**: This provides a detailed JSON output containing all the
    low-level configuration and state information about the container,
    similar to the `podman`{.inlineCode}` inspect`{.inlineCode} command.

-   **Terminal**: This is one of the most powerful
    features.[]{.sentence-end} It gives you direct shell access inside
    the running container.[]{.sentence-end} You can use it to explore
    the container\'s filesystem, check running processes, or modify
    configuration files for temporary testing.[]{.sentence-end}

    <figure class="mediaobject-one">
    <img src="images/B31467_15_7.png"
    style="width:460.79999999999995px; height:239.4550218340611px;"
    alt="Image 7" />
    </figure>

    Figure 15.7: The Details view of the running NGINX container,
    showing the Logs tab

## Attaching persistent storage using volumes {#Chapter_15.xhtml#h2_359 .heading-2}

Containers are[]{#Chapter_15.xhtml#idx_5745b848 .index-entry
index-entry="persistent storage:attaching, with volumes"} ephemeral by
nature.[]{.sentence-end} If we were to remove
[]{#Chapter_15.xhtml#idx_4a3ea030 .index-entry
index-entry="volumes:persistent storage, attaching"}our
`my-web-server`{.inlineCode} container, any data written inside it would
be lost.[]{.sentence-end} To persist data, we use
volumes.[]{.sentence-end} Let\'s create a volume to store the NGINX web
content and attach it to a new container:

1.  Navigate to the **Volumes** section from the left
    sidebar.[]{.sentence-end} This screen lists all available volumes.

2.  Click the **Create** button.

3.  In the dialog that appears, provide a name for your volume, for
    example, `web-content`{.inlineCode}, and click
    **Create**.[]{.sentence-end} The new volume will now appear in the
    list.[]{.sentence-end}

    Now, let\'s create a new NGINX container and attach this volume to
    it.

4.  Go back to the **Images** section, find the `nginx`{.inlineCode}
    image, and click the **Run** icon again.

5.  On the **Create a container** screen, give it a new name such as
    `persistent-web-server`{.inlineCode} and configure the port mapping
    again, but with a different host port (`8081:80`{.inlineCode}).

6.  Scroll down to the **Volumes** section of the configuration screen.

7.  Here, you will map the `web-content`{.inlineCode} volume we just
    created to the directory where NGINX stores its default HTML files,
    which is
    `/`{.inlineCode}`usr`{.inlineCode}`/share/`{.inlineCode}`nginx`{.inlineCode}`/html`{.inlineCode}:

    -   In the first field, paste the `web-content`{.inlineCode} volume
        path that you previously created.[]{.sentence-end} You can get
        the volume\'s path in the **Volumes** section, in the volume\'s
        details.
    -   In the []{#Chapter_15.xhtml#idx_105d0599 .index-entry
        index-entry="persistent storage:attaching, with volumes"}second
        field, enter the[]{#Chapter_15.xhtml#idx_fed52167 .index-entry
        index-entry="volumes:persistent storage, attaching"} container
        path:
        `/`{.inlineCode}`usr`{.inlineCode}`/share/`{.inlineCode}`nginx`{.inlineCode}`/html`{.inlineCode}.

    <figure class="mediaobject-one">
    <img src="images/B31467_15_8.png"
    style="width:460.79999999999995px; height:239.53983471074378px;"
    alt="Image 8" />
    </figure>

    Figure 15.8: The Volumes mapping section within the Create a
    container screen, showing the web-content volume mapped to
    /usr/share/nginx/html

8.  Click **Start Container**.[]{.sentence-end} Now, this new container
    is using the `web-content`{.inlineCode} volume to
    serve[]{#Chapter_15.xhtml#idx_6e4e1add .index-entry
    index-entry="persistent storage:attaching, with volumes"} its
    files.[]{.sentence-end} Even if you stop
    and[]{#Chapter_15.xhtml#idx_5586da43 .index-entry
    index-entry="volumes:persistent storage, attaching"} remove this
    container, the volume and its data will remain.[]{.sentence-end} You
    can then launch another container and attach it to the same volume
    to resume using the persisted data.

## Working with container networking {#Chapter_15.xhtml#h2_360 .heading-2}

Networking is a[]{#Chapter_15.xhtml#idx_7a0541ce .index-entry
index-entry="container networking"} fundamental aspect of
containerization.[]{.sentence-end} As we\'ve already seen, Podman
Desktop makes port mapping incredibly simple during container
creation.[]{.sentence-end} This is the most common networking
interaction you\'ll have, allowing you to expose services running inside
containers to your host machine or the wider network.

While Podman supports the creation of complex custom networks for
multi-container applications, the Podman Desktop interface focuses on
the essentials.[]{.sentence-end} During container creation, you can
assign a container to any pre-existing network via the **Networking**
dropdown in the advanced settings.[]{.sentence-end} However, creating
and managing the networks themselves is typically an advanced operation
handled via the command line.

For day-to-day use, the key networking information is readily available
in the container\'s **Details** view.[]{.sentence-end} The **Summary**
tab clearly displays any port mappings, allowing you to quickly see
which host port corresponds to which container port, saving you from
having to inspect the container\'s configuration manually.

With these features, Podman Desktop provides a robust and intuitive
toolkit for handling the entire container lifecycle, from creation and
management to persistence and networking, all without needing to touch
the command line.[]{.sentence-end} We are now ready to explore its
user-friendly interface for managing a real Kubernetes cluster in the
next section.

# Connecting to and interacting directly with Kubernetes clusters {#Chapter_15.xhtml#h1_361 .heading-1}

Before diving into the graphical interface, it\'s helpful to understand
the role of Kubernetes.[]{.sentence-end} Born out of Google\'s internal
need to manage applications at an immense scale, Kubernetes is an open
source container orchestration platform that automates the deployment,
scaling, and management of containerized applications.[]{.sentence-end}
It groups containers that make up an application into logical units
called **Pods** for[]{#Chapter_15.xhtml#idx_7bd4e35e .index-entry
index-entry="Pods"} easy management and discovery.[]{.sentence-end} Over
the years, Kubernetes has evolved into the de facto standard for
multi-node container orchestration, supported by a vast ecosystem and a
vibrant community.[]{.sentence-end} Its power lies in its declarative
nature: you define the desired state of your application, and Kubernetes
works to maintain that state, handling failures, scaling, and service
discovery automatically.

Podman Desktop embraces this standard by providing a seamless, built-in
client to connect to and manage any Kubernetes cluster.

<figure class="mediaobject">
<img src="images/B31467_15_9.png"
style="width:528.0px; height:274.4727272727273px;" alt="Image 9" />
</figure>

Figure 15.9: The Kubernetes Settings screen in Podman Desktop, showing
the empty list of available Kubernetes contexts

This turns Podman Desktop into a comprehensive control panel, allowing
you to manage your local Podman containers and your remote Kubernetes
workloads from a single, unified interface.

## Connecting to a local cluster with minikube {#Chapter_15.xhtml#h2_362 .heading-2}

For developers[]{#Chapter_15.xhtml#idx_02a9a850 .index-entry
index-entry="minikube:used, for connecting to local cluster"} who prefer
to run a Kubernetes cluster directly on their local machine,
**minikube** is an excellent and widely adopted tool.[]{.sentence-end}
It\'s a project designed to spin up a single-node Kubernetes cluster
inside a virtual machine or a container on your personal
computer.[]{.sentence-end} This approach is perfect for learning
Kubernetes, daily development, and testing your applications in a
lightweight, self-contained environment.

You can find all the information and the documentation on how to set up
a `minikube`{.inlineCode} instance in *[Chapter
14](#Chapter_14.xhtml#h1_330){.chapref}*, *Interacting with* *systemd*
*and* *Kubernetes*.

## Connecting Podman Desktop to a Kubernetes cluster {#Chapter_15.xhtml#h2_363 .heading-2}

Let\'s see now[]{#Chapter_15.xhtml#idx_afdc90bf .index-entry
index-entry="Podman Desktop:connecting, to Kubernetes cluster"} how to
connect an existing []{#Chapter_15.xhtml#idx_97b5c090 .index-entry
index-entry="Kubernetes cluster:Podman Desktop, connecting to"}Kubernetes
cluster to Podman Desktop:

1.  The first step to managing a Kubernetes cluster is telling Podman
    Desktop where to find it.[]{.sentence-end} This is handled through a
    `kubeconfig`{.inlineCode} file, which contains the credentials and
    endpoint information for one or more clusters.[]{.sentence-end}
    Podman Desktop automatically detects `kubeconfig`{.inlineCode} files
    from their default location
    (`~/.`{.inlineCode}`kube`{.inlineCode}`/`{.inlineCode}`config`{.inlineCode}).

2.  To view and manage your Kubernetes connections, look at the bottom
    of the Podman Desktop window.[]{.sentence-end} You will see a button
    indicating the current context (it might say **Podman** by
    default).[]{.sentence-end} Clicking this button reveals a menu of
    all detected Kubernetes contexts.

3.  Alternatively, navigate to **Settings** \|
    **Kubernetes**.[]{.sentence-end} This screen provides a detailed
    list of all available Kubernetes contexts found on your
    system.[]{.sentence-end}

    <figure class="mediaobject-one">
    <img src="images/B31467_15_10.png"
    style="width:460.79999999999995px; height:239.53983471074378px;"
    alt="Image 10" />
    </figure>

    Figure 15.10: The Kubernetes main screen in Podman Desktop, showing
    the default available context from the kubeconfig file

4.  From the **Settings** list, you can select the context you wish to
    interact with.[]{.sentence-end} Let\'s assume you have a
    `minikube`{.inlineCode} cluster running, as set up in the previous
    chapter.[]{.sentence-end} Select the `minikube`{.inlineCode} context
    from the list.

5.  Once []{#Chapter_15.xhtml#idx_291fdc40 .index-entry
    index-entry="Podman Desktop:connecting, to Kubernetes cluster"}selected,
    Podman Desktop\'s []{#Chapter_15.xhtml#idx_05d2ea64 .index-entry
    index-entry="Kubernetes cluster:Podman Desktop, connecting to"}entire
    UI will switch its focus from the local Podman engine to the remote
    `minikube`{.inlineCode} cluster.[]{.sentence-end} The sidebar will
    now be populated with Kubernetes resources such as Pods,
    Deployments, Services, and so on.

As shown, the process is really straightforward, and it could make
Podman a good companion for managing any existing Kubernetes cluster.

### Creating a local kind cluster {#Chapter_15.xhtml#h3_364 .heading-3}

**Kubernetes** **in** **Docker** (**kind**) is
a[]{#Chapter_15.xhtml#idx_6c4bdcff .index-entry
index-entry="Kubernetes in Docker (kind)"} great alternative to
`minikube`{.inlineCode} for developers aiming to work on a local
Kubernetes cluster.[]{.sentence-end} Different from
`minikube`{.inlineCode}, `k`{.inlineCode}`ind`{.inlineCode} doesn\'t
create a local []{#Chapter_15.xhtml#idx_1c9203cb .index-entry
index-entry="local kind cluster:creating"}virtual machine but totally
relies on the container engine to spin up all the Kubernetes control
plane services.[]{.sentence-end} It\'s a very simple and intuitive
approach, and great for working on local application development.

To create a `k`{.inlineCode}`ind`{.inlineCode} cluster with Podman
Desktop, go to **Settings** \| **Resources** and click on the **Create
new...** button in the **Kind** box.

<figure class="mediaobject">
<img src="images/B31467_15_11.png"
style="width:528.0px; height:288.43636363636364px;" alt="Image 11" />
</figure>

Figure 15.11: Kind cluster creation

The next screen allows users to customize the name of the
`kind`{.inlineCode} cluster (default is **kind-cluster**), the provider
type (default is **Podman**), the default HTTP/HTTPS ports, and to
define the Ingress controller setup.

<figure class="mediaobject">
<img src="images/B31467_15_12.png"
style="width:528.0px; height:288.43636363636364px;" alt="Image 12" />
</figure>

Figure 15.12: Kind cluster details definition

Click on the **Create** button[]{#Chapter_15.xhtml#idx_57732518
.index-entry index-entry="local kind cluster:creating"} and wait for the
`k`{.inlineCode}`ind`{.inlineCode} cluster to start.[]{.sentence-end}
When the cluster creation is completed, you will see a successful
creation screen:

<figure class="mediaobject">
<img src="images/B31467_15_13.png"
style="width:493.76250000000005px; height:269.7330681818182px;"
alt="Image 13" />
</figure>

Figure 15.13: Kind cluster creation in progress

After creation, the `k`{.inlineCode}`ind`{.inlineCode} cluster will be
visible under the Kubernetes resources.

To stop the `k`{.inlineCode}`ind`{.inlineCode} cluster, go
[]{#Chapter_15.xhtml#idx_ba3d1280 .index-entry
index-entry="local kind cluster:creating"}to **Settings** \|
**Resources** and simply click on the **Stop** icon.[]{.sentence-end} To
start it again, click on the **Play** icon.

<figure class="mediaobject">
<img src="images/B31467_15_14.png"
style="width:528.0px; height:288.43636363636364px;" alt="Image 14" />
</figure>

Figure 15.14: Kind cluster available in the Resources section

After completing the process, we will get a fully functional Kubernetes
cluster that we can use for testing purposes.[]{.sentence-end} Let\'s
now see how to explore and manage Kubernetes resources.

## Exploring and managing Kubernetes resources {#Chapter_15.xhtml#h2_365 .heading-2}

With Podman Desktop connected to your cluster, you can now browse and
manage its resources as easily as you manage local containers.

### Managing Pods {#Chapter_15.xhtml#h3_366 .heading-3}

Pods are the smallest deployable[]{#Chapter_15.xhtml#idx_c71c67d6
.index-entry index-entry="Pods:managing"} units of computing in
Kubernetes, representing a group of one or more
containers.[]{.sentence-end} Follow these steps:

1.  Navigate to the **Pods** section using the left
    sidebar.[]{.sentence-end} You will see a list of all Pods running
    across all namespaces in your cluster, if any.

2.  You can use the **Namespace** dropdown at the top of the list to
    filter the view and focus on a specific namespace, such as
    `default`{.inlineCode} or
    `kube`{.inlineCode}`-system`{.inlineCode}.[]{.sentence-end} The list
    provides at-a-glance information, including the Pod\'s name, status
    (e.g., **Running**, **Pending**, or **Error**), and
    []{#Chapter_15.xhtml#idx_42871196 .index-entry
    index-entry="Pods:managing"}the namespace it belongs
    to.[]{.sentence-end}

    <figure class="mediaobject-one">
    <img src="images/B31467_15_15.png"
    style="width:460.79999999999995px; height:239.53983471074378px;"
    alt="Image 15" />
    </figure>

    Figure 15.15: The Pods list view in Podman Desktop, filtered to a
    specific namespace and showing several running Pods

3.  To get more details, click on any Pod in the list.[]{.sentence-end}
    This opens a **Details** view, very similar to the one for local
    containers:
    -   **Logs**: This tab allows you to view the real-time logs of the
        containers running inside the Pod.[]{.sentence-end} If the Pod
        has multiple containers, you can select which container\'s logs
        you want to see.
    -   **Inspect**: This tab shows the low-level details of the Pod
        resource in JSON format.
    -   **Terminal**: This []{#Chapter_15.xhtml#idx_a9328105
        .index-entry index-entry="Pods:managing"}powerful feature lets
        you open an interactive shell directly into a running container
        within the Pod, which is incredibly useful for debugging.

### Managing deployments and services {#Chapter_15.xhtml#h3_367 .heading-3}

While Pods are the basic building[]{#Chapter_15.xhtml#idx_0b27242b
.index-entry index-entry="deployments:managing"} blocks, in a real-world
scenario, you typically manage[]{#Chapter_15.xhtml#idx_e581f021
.index-entry index-entry="services:managing"} them through higher-level
objects such as *Deployments* and expose them with *Services*.

1.  Navigate to the **Deployments** section in the
    sidebar.[]{.sentence-end} Here, you\'ll see a list of all
    Deployments in the cluster.[]{.sentence-end} This view shows the
    desired number of replicas versus the currently available ones,
    giving you a quick check on the health of your applications.

2.  Click on a Deployment to see its details, including the Pods it
    manages.[]{.sentence-end} From here, you can perform basic actions
    such as deleting the deployment, which will also terminate its
    associated Pods.

3.  Next, navigate to the **Services** section.[]{.sentence-end} A
    Service in Kubernetes is an abstraction that defines a logical set
    of Pods and a policy by which to access them.[]{.sentence-end} This
    list shows all the services, their type (`ClusterIP`{.inlineCode},
    `NodePort`{.inlineCode}, or `LoadBalancer`{.inlineCode}), and the
    ports they expose.[]{.sentence-end}


    ![](images/B31467_15_16.png)


    Figure 15.16: The Services list view, showing services of different
    types, including a NodePort service with its assigned port

Let\'s put this into a[]{#Chapter_15.xhtml#idx_e5b88426 .index-entry
index-entry="deployments:managing"} practical context.[]{.sentence-end}
Imagine that our WordPress application[]{#Chapter_15.xhtml#idx_5066d415
.index-entry index-entry="services:managing"} from the previous chapter
is running on the cluster:

-   In the **Deployments** list, you would find the
    `wordpress`{.inlineCode}`-pod`{.inlineCode} and
    `mysql`{.inlineCode}`-pod`{.inlineCode} Deployments.
-   In the **Services** list, you would see the
    `wordpress`{.inlineCode}`-pod`{.inlineCode} Service of the
    `NodePort`{.inlineCode} type.[]{.sentence-end} By looking at its
    details, you could find the specific port to access the WordPress
    setup page through your cluster\'s IP address.
-   If a WordPress Pod was having issues, you could go to the **Pods**
    list, find the specific Pod, and use the **Logs** tab to check for
    errors or the **Terminal** tab to `exec`{.inlineCode} into the
    container and check its configuration files.

By integrating Kubernetes management so deeply, Podman Desktop
streamlines the developer workflow.[]{.sentence-end} It eliminates the
need to constantly switch between a terminal for `kubectl`{.inlineCode}
commands and other tools.[]{.sentence-end} Instead, it provides a
centralized, intuitive interface to observe, debug, and interact with
applications, whether they are running locally on Podman or remotely on
a full-fledged Kubernetes cluster.

We are now ready to explore its user-friendly interface for managing AI
workloads in the next section.

# Deploying local AI inference for app integration


##  Podman AI Lab

Recognizing the trend toward local AI, the creators of Podman introduced **Podman**
**AI Lab**, an integrated extension within Podman Desktop. The primary goal of Podman AI Lab is to radically simplify the process of downloading, running, and managing AI models on a developer\'s machine. It abstracts away the complex, manual steps of setting up environments, configuring model servers, and managing dependencies. Instead, it leverages the power of containers to provide a one-click experience for spinning up a local inference server, complete with a ready-to-use API endpoint for your applications.

Key features include the following:
-   A curated catalog of popular, open source models
-   Simplified model download and setup
-   Automatic provisioning of a containerized inference server
-   Clear presentation of API endpoints for integration

## Enabling Podman AI Lab

Podman AI Lab is included as an extension and needs to be enabled before you can use it:

1.  Open Podman Desktop and select **Extensions**.
2.  You will see a list of available extensions. Locate **Podman** **AI Lab**.
3.  Click the toggle switch to enable it. The application might require a quick restart to apply the
    changes.

    ![](images/B31467_15_17.png)

Once enabled, a new **AI Lab** icon will appear in your left sidebar, ready for you to explore.

## Running your first AI model

Download a model and run a local inference server. We will use a small, efficient model for this example, such as `Phi-3-mini`, which is
designed to run well on consumer hardware:

1.  Click on the new **AI Lab** icon in the sidebar. This will take you to the **AI Lab** dashboard. Then, click on the **Catalog** section, which presents a list of available models you can download.

2.  Find the model you wish to run (e.g., **Phi-2-GGUF**) from the catalog and click on its card to see more details. We are choosing this model for its reduced footprint.

3.  On the catalog\'s main screen, you will see a **Download** icon on the right side. Click it to start downloading the model files to your machine. This may take some time, depending on the model's size and your internet connection.

4.  Once the download is complete, the button will change to a *rocket* icon to create the model service. Click it to begin setting up the inference server.

5.  A configuration dialog will appear. Here, you can accept the defaults or customize the setup. For now, the default settings are sufficient. Click **Create service** to proceed.

    ![](images/B31467_15_18.png)

If you see any warning regarding the available RAM, please disregard it if you are running Podman Desktop on a Linux machine; otherwise, please follow any instructions to adjust the memory settings.

Podman AI Lab will now work in the background to provision an inference server. This is where the magic
happens: it pulls the `ramalama` specialized container image, starts a container, loads the model into it, and exposes the necessary ports.

**RamaLama** is an open source tool designed to simplify running and serving AI models on a local machine by using the familiar workflow of OCI containers. It automatically detects the user\'s hardware, such as GPUs from NVIDIA, AMD, or Apple, and pulls the correct accelerated container image, eliminating the need for complex host system configuration. This allows engineers to manage
AI models with container-centric commands such as `run`, `pull`, and `serve`, while also enhancing security by running the models in isolated, rootless containers with no network access by default. The project supports pulling models from multiple registries, including Hugging Face and Ollama, and allows users to interact with them through a REST API or as a chatbot.

Once it has pulled the inference server container image, Podman Desktop will run it.

## Verifying the running model server

How can we be sure of what AI Lab has done? We can use the core container management features of Podman Desktop to see the result:

1.  Navigate to the **Containers** section in the left sidebar.

2.  You will see a new container running. Its name will be related to the AI Lab service (e.g., `ramalama-llama-server`. This confirms that AI Lab simply orchestrated the creation of a standard Podman container to host the model.

    ![](images/B31467_15_19.png)

By clicking on this container, you can inspect it just like any other: view its logs, check the mapped ports, or even open a terminal inside it. This transparency is a key benefit because you are always in control and can use your existing container skills to manage the underlying infrastructure.

## Obtaining the API endpoint

Now that the server is running, we need the endpoint to interact with it from our own applications:

1.  Return to the **AI Lab** dashboard.

2.  Go to the **Services** tab, and you will see that your **Phi-2-GGUF** model now has a **Running** status.

3.  Click on the model to view its details. The interface will now display a ready-to-use **API Endpoint URL**. This is typically an OpenAI-compatible endpoint, making it easy to integrate with a vast range of existing libraries and tools. The URL will look something
    like http://localhost:8080/v1/chat/completions

![](images/B31467_15_20.png)

You can test the endpoint immediately. Copy the URL, open a terminal on your host machine, and use a tool such as `curl` to send a
request.[]{.sentence-end} While we will build a full application in the
next section, a simple test demonstrates that your local AI server is
live and ready for integration.

With just a few clicks, Podman AI Lab has provisioned a fully
functional, containerized AI inference server on your local
machine.[]{.sentence-end} This powerful capability empowers you to
experiment with and develop AI-driven features with unparalleled ease,
all while maintaining privacy and control over your
data.[]{.sentence-end} Thanks to these great features, we are now ready
to explore how it could be easy to develop a new end-to-end use case
using Podman Desktop in the next section.

# Integrating an AI model running on Podman for building your own chatbot use case {#Chapter_15.xhtml#h1_374 .heading-1}

We have reached the final and most rewarding part of our
journey.[]{.sentence-end} So far, we have provisioned a local AI
inference server.[]{.sentence-end} Now, we will put it to work by
building a complete, end-to-end application: a private, personal chatbot
that runs entirely on your machine.[]{.sentence-end} This section will
guide you through integrating your containerized AI model with a
powerful, open source user interface, demonstrating the true value of AI
when harnessed by a real-world use case.

An AI model, no matter how powerful, is of little use on its
own.[]{.sentence-end} Its value is unlocked when it is integrated into
an application that solves a problem or provides a
service.[]{.sentence-end} The true potential of generative AI is
realized in these end-to-end use cases, which inspire innovation and
showcase its practical benefits.[]{.sentence-end} With Podman and Podman
Desktop, we have the unique ability to run not only the AI model but
also the application that uses it, side by side, in a secure and
manageable containerized environment.

In this section, we will build a fully functional chatbot using an AI
model of our choice and integrate it with **AnythingLLM**, a
[]{#Chapter_15.xhtml#idx_1df72259 .index-entry
index-entry="AnythingLLM"}popular open source web interface designed to
create a custom personal assistant.[]{.sentence-end} The entire
stack---from the AI model to the web application---will be powered by
Podman Desktop and its AI Lab extension.

## Step 1: Starting an OpenAI-compatible inference server {#Chapter_15.xhtml#h2_375 .heading-2}

AnythingLLM requires a[]{#Chapter_15.xhtml#idx_95e23bb4 .index-entry
index-entry="OpenAI-compatible inference server:starting"} connection to
an LLM that exposes an OpenAI-compatible API.[]{.sentence-end}
Fortunately, Podman AI Lab\'s inference servers are designed for this
exact purpose.

Let\'s start by ensuring that our model server is up and
running.[]{.sentence-end} Please be sure to have a model running from
the previous section, so you can proceed to the next step.

## Step 2: Running the AnythingLLM application container
Next, we need to run the AnythingLLM web application itself. We will do this by pulling its official container image and running it using Podman Desktop:

1.  Navigate to the **Images** section in the sidebar.
2.  Click the **Pull an image** button. In the dialog, enter the official image name for AnythingLLM
    `mintplexlabs/anythingllm` and click **Pull image**.
3.  After the image is downloaded, find it in the list and click the **Run** icon to start the container creation process.
4.  The **Create a container** screen will appear. We need to configure two important settings
    for AnythingLLM to work correctly:
    -   **Port** **mapping**: The AnythingLLM web server listens on port `3001` inside the container. Map
        port `3001`{.inlineCode} on your host to port `3001` in the container.
    -   **Volume for** **persistent** **storage**: To ensure that your chat history and configuration are saved, we need to attach a
        volume. The application stores its data in the `/app/server/storage` directory:
        - First, go to the **Volumes** section in Podman Desktop and create a new volume named `anythingllm-storage`
        - Now, we need to create an empty`.env` file in that storage location. From the just-created volume, you can inspect the details of the underlying directory and run the `touch`  command for creating the empty file. Finally, also apply the right permission to the root folder. The command will look like this:

        ``` {.programlisting .snippet-con-two}
        $ touch /home/seal/.local/share/containers/storage/volumes/anythingllm/_data/.env
        $ chmod -R 777 /home/seal/.local/share/containers/storage/volumes/anythingllm/_data
        ```

        - Return to the container creation screen. In the **Volumes** section, map the `anythingllm-storage` volume to the container path, `/app/server/storage`.
        - In the **Volumes** section, also map the `.env`{.inlineCode} file we created with the `.env` file inside the container available at this location, `/app/server/.env`
        - Finally, append the `:z` suffix to the container path strings. This ensures that SELinux grants the container the necessary permissions to write to the system storage.
        - Define the `STORAGE_DIR` environment variable with the `/app/server/storage` value.
        - Add the required capabilities in the **Security** tab: `SYS_ADMIN`

5.  Give the container a name, such as `anything-llm`, and click **Start Container**.

    ![](images/B31467_15_21.png)


After that, the container will start, and we will be ready to connect it to our model!

## Step 3: Connecting the model to the application

With both our AI model and web application running in separate containers, the final step is to connect them:

1.  First, we need the inference URL for our model:
    1.  Navigate to the **AI Lab** section and click on your running model (**Phi-2-GGUF**).
    2.  On its **Details** page, copy the API endpoint
        URL. It will look like http://localhost:43717/v1.

2.  Now, open a web browser and navigate to the **AnythingLLM** interface at http://localhost:3001

![](images/B31467_15_22.png)


3.  You will be guided through a one-time setup process.When you reach the **LLM Preference** step, you will configure it to use your local model:
    -   **LLM** **Provider**: select **Generic OpenAI**.
    -   **Chat Endpoint**: This is where you need to paste the inference server URL. Since the model is served through a
        container but it\'s also exposed on the host machine, we could use this special DNS name to reach it. Modify
        the URL to be something like `http://host.containers.internal:43717/v1`
    -   **API Key**: For local models served this way, an API key is not required. You can leave this field blank.
    -   **Chat Model Name**: Enter the identifier for your model as shown in the AI Lab catalog, such as `/models/phi-2.Q4_K_M.gguf` You can get your model name by running this simple `curl` command on your host:
        ```
        $ curl -H 'accept: application/json' -X 'GET' 'http://localhost:43717/v1/models'
        ```

    -   **Token context window**: Enter the value for the maximum number of tokens (roughly words/subword units) that the model can
        \"see\" at once. You can start with a conservative `4096` value and increase later if necessary.

![](images/B31467_15_23.png)
        


4.  Complete the setup. You will be taken to the main chat interface.

## Step 4: Testing your personal assistant

You are now ready to interact with your fully private, locally-hosted chatbot.

First of all, select the first created workspace through the left  navigation bar. Then, try asking a question, such as
`What is Podman?`

The request will travel from the AnythingLLM container to your local inference server container, be processed by the AI model, and the
response will be streamed back to the web interface.

To confirm that the entire stack is running as expected, return to Podman Desktop and navigate to the **Containers** list.

You should now see at least two running containers: your AI inference server and your web application `anythingllm`.

![](images/B31467_15_24.png)





# Further reading {#Chapter_15.xhtml#h1_380 .heading-1}

-   Podman Desktop downloads:
    [[https://podman-desktop.io/downloads]{.url}](https://podman-desktop.io/downloads){style="text-decoration: none;"}
-   Podman Desktop tutorials:
    [[https://podman-desktop.io/tutorial]{.url}](https://podman-desktop.io/tutorial){style="text-decoration: none;"}
-   Podman Desktop blog:
    [[https://podman-desktop.io/blog]{.url}](https://podman-desktop.io/blog){style="text-decoration: none;"}
-   Podman Desktop AI LAB:
    [[https://podman-desktop.io/docs/ai-lab]{.url}](https://podman-desktop.io/docs/ai-lab){style="text-decoration: none;"}
-   RamaLama project:
    [[https://ramalama.ai/]{.url}](https://ramalama.ai/){style="text-decoration: none;"}
-   The `minikube`{.inlineCode} project\'s home page:
    [[https://minikube.sigs.k8s.io/]{.url}](https://minikube.sigs.k8s.io/){style="text-decoration: none;"}
-   Podman community links:
    [[https://podman.io/community/]{.url}](https://podman.io/community/){style="text-decoration: none;"}
