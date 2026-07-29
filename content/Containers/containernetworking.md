# 12 {#Chapter_12.xhtml#h1_295 .chapterNumber}

# Implementing Container Networking Concepts {#Chapter_12.xhtml#h1_296 .chapterTitle}

Container network isolation leverages network namespaces to provide
separate network stacks for each container.[]{.sentence-end} Without a
container runtime, managing network interfaces across multiple
namespaces would be complex.[]{.sentence-end} Podman provides flexible
network management that allows users to customize how containers
communicate with external and other containers inside the same host.

In this chapter, we will learn about the common configuration practices
for managing container networking, along with the differences between
rootless and rootful containers.

In this chapter, we\'re going to cover the following main topics:

-   Container networking and Podman setup
-   Interconnecting two or more containers
-   Exposing containers outside our underlying host
-   Rootless container network behavior

# Technical requirements {#Chapter_12.xhtml#h1_297 .heading-1}

To complete this chapter, you will need a machine with a working Podman
installation.[]{.sentence-end} As we mentioned in *[Chapter
3](#Chapter_3.xhtml#h1_83){.chapref}*, *Running* *the First Container*,
all the examples in this book can be executed on a Fedora 40 system or
later, but can be reproduced on your operating system of
choice.[]{.sentence-end} The examples in this chapter will be related to
Podman v5.x.y and its default network backend.

A good understanding of the topics that were covered in *[Chapter
4](#Chapter_4.xhtml#h1_114){.chapref}*, *Managing Running Containers*,
*[Chapter 5](#Chapter_5.xhtml#h1_133){.chapref}*, *Implementing*
*Storage for the Container\'s Data*, and *[Chapter
9](#Chapter_9.xhtml#h1_219){.chapref}*, *Pushing Images to a Container*
*Registry*, will help you grasp the container networking topics we\'ll
be covering.

You must also have a good understanding of basic networking concepts to
understand topics such as routing, the IP protocol, DNS, and
firewalling.

# Configuring container networking and setting up Podman {#Chapter_12.xhtml#h1_298 .heading-1}

In this section, we\'ll cover[]{#Chapter_12.xhtml#idx_9caf0586
.index-entry index-entry="container networking:configuring"} Podman\'s
networking implementation and how to configure
networks.[]{.sentence-end} Podman 4 introduced an important change to
the network stack, which is why, considering that Podman 5 has already
been available for a while, we are going to showcase only examples based
on version 5\'s implementation.

Just for your reference, Podman 3 and 4
leveraged[]{#Chapter_12.xhtml#idx_84c95028 .index-entry
index-entry="Container Network Interface (CNI)"} the **Container Network
Interface** (**CNI**) to manage local networks that are created on the
host.[]{.sentence-end} The CNI provides a standard set of specifications
and libraries to create and configure plugin-based network interfaces in
a container environment.

CNI specifications were created for Kubernetes to provide a network
configuration format that\'s used by the
container[]{#Chapter_12.xhtml#idx_3e156b02 .index-entry
index-entry="Podman:setting up"} runtime to set up the defined plugins,
as well as an execution protocol between plugin binaries and
runtimes.[]{.sentence-end} The advantage of this plugin-based approach
is that vendors and communities can develop third-party plugins that
satisfy the CNI\'s specifications.[]{.sentence-end} The Podman 4 network
stack is instead based on a new project called Netavark, a
container-native networking implementation completely written in Rust
and designed to work with Podman.[]{.sentence-end}
Rust[]{#Chapter_12.xhtml#idx_acd6e221 .index-entry index-entry="Rust"}
is a great programming language for developing system and network
components, thanks to its efficient memory management and high
performance, similar to the C programming language.[]{.sentence-end}
Netavark provides better support for dual-stack networking (IPv4/IPv6)
and inter-container DNS resolution, along with a tighter bond with the
Podman project development roadmap.

Starting from Podman 5, CNI support has been deprecated, and it will be
removed from Podman 6, which is why we are not going to deep dive into
any use case or configuration leveraging the deprecated
and[]{#Chapter_12.xhtml#idx_7882907f .index-entry
index-entry="Podman:setting up"} removed CNI network backend.

In the next subsection, we\'ll focus on the Netavark backend, which is
used by Podman 5 to orchestrate container networking.

## Configuring Netavark {#Chapter_12.xhtml#h2_299 .heading-2}

Podman 4\'s release introduced []{#Chapter_12.xhtml#idx_e7e240b7
.index-entry index-entry="Netavark:configuring"}Netavark as the default
network backend.[]{.sentence-end} The advantages of Netavark are as
follows:

-   Support for dual IPv4/IPv6 stacks
-   Support for DNS native resolution using the
    `aardvark-dns`{.inlineCode} companion project
-   Support for rootless containers
-   Support for different firewall implementations, even though the
    suggested one is `nftables`{.inlineCode}

Netavark uses JSON format to configure networks; files are stored under
the `/etc/containers/networks`{.inlineCode} path for rootful containers
and the `~/.local/share/containers/storage/networks`{.inlineCode} path
for rootless containers.

The following configuration file shows an example network that\'s been
created and managed under Netavark:

``` {.programlisting .snippet-code}
[
     {
          "name": "netavark-example",
          "id": "d98700453f78ea2fdfe4a1f77eae9e121f3cbf4b6160dab89edf9ce23cb924d7",
          "driver": "bridge",
          "network_interface": "podman1",
          "created": "2025-02-17T21:37:59.873639361Z",
          "subnets": [
               {
                    "subnet": "10.89.4.0/24",
                    "gateway": "10.89.4.1"
               }
          ],
          "ipv6_enabled": false,
          "internal": false,
          "dns_enabled": true,
          "ipam_options": {
               "driver": "host-local"
          }
     }
]
```

The first noticeable element is []{#Chapter_12.xhtml#idx_5d2c0d9e
.index-entry index-entry="Netavark:configuring"}the compact size of the
configuration file.[]{.sentence-end} The following fields are defined:

-   `name`{.inlineCode}: This is the name of the network.
-   `id`{.inlineCode}: This is the unique network ID.
-   `driver`{.inlineCode}: This specifies the kind of network driver
    that\'s being used.[]{.sentence-end} The default is
    `bridge`{.inlineCode}.[]{.sentence-end} Netavark also supports
    MACVLAN drivers.
-   `network_interface`{.inlineCode}: This is the name of the network
    interface associated with the network.[]{.sentence-end} If
    `bridge`{.inlineCode} is the configured driver, this will be the
    name of the Linux bridge.[]{.sentence-end} In the preceding example,
    a bridge is created called `podman1`{.inlineCode}.
-   `created`{.inlineCode}: This is the network creation timestamp.
-   `subnets`{.inlineCode}: This provides a list of subnet and gateway
    objects.[]{.sentence-end} Subnets are assigned
    automatically.[]{.sentence-end} However, when you\'re creating a new
    network with Podman, users can provide a custom
    CIDR.[]{.sentence-end} Netavark allows you to manage multiple
    subnets and gateways on a network.
-   `ipv6_enabled`{.inlineCode}: Native support for IPv6 in Netavark can
    be enabled or disabled with this Boolean.
-   `internal`{.inlineCode}: This Boolean is used to configure a network
    for internal use only and to block external routing.
-   `dns_enabled`{.inlineCode}: This Boolean enables DNS resolution for
    the network and is served by the `aardvark-dns`{.inlineCode} daemon.
-   `ipam_options`{.inlineCode}: This object defines a series of
    `ipam`{.inlineCode} parameters.[]{.sentence-end} In the preceding
    example, the only option is the kind of IPAM driver,
    `host-local`{.inlineCode}, which behaves in a way similar to the CNI
    `host-local`{.inlineCode} plugin.

The default Podman 4[]{#Chapter_12.xhtml#idx_b4a46d28 .index-entry
index-entry="Netavark:configuring"} network, named
`podman`{.inlineCode}, implemented a `bridge`{.inlineCode} driver (the
bridge\'s name is `podman0`{.inlineCode}).[]{.sentence-end} Netavark is
also an executable binary that\'s installed by default in the
`/usr/libexec/podman/netavark`{.inlineCode} path.[]{.sentence-end} It
has a simple **command-line interface** (**CLI**)
that[]{#Chapter_12.xhtml#idx_311d8aa4 .index-entry
index-entry="command-line interface (CLI)"} implements the
`setup`{.inlineCode} and `teardown`{.inlineCode} commands, applying the
network configuration to a given network namespace (see
`man netavark`{.inlineCode}).[]{.sentence-end} Consider that users are
never expected to run Netavark themselves.[]{.sentence-end} Podman does
not provide documentation on how to do so, and deliberately installs
into a path not in the user\'s default `$PATH`{.inlineCode}.

Now, let\'s look at the effects of creating a new container with
Netavark.

## Creating a container with Podman and Netavark {#Chapter_12.xhtml#h2_300 .heading-2}

Netavark manages the creation of network configurations in the container
network namespace and the host network namespace, including the creation
of `veth`{.inlineCode} pairs and the Linux bridge that\'s defined in the
config file.

Before the first container is created
in[]{#Chapter_12.xhtml#idx_28a4a2df .index-entry
index-entry="containers:creating, with Podman"} the default Podman
network, no bridges are[]{#Chapter_12.xhtml#idx_7c9e318a .index-entry
index-entry="containers:creating, with Netavark"} created, and the host
interfaces are the []{#Chapter_12.xhtml#idx_abc1ed9e .index-entry
index-entry="Podman:container, creating"}only ones available, along with
the loopback[]{#Chapter_12.xhtml#idx_9ced9c8b .index-entry
index-entry="Netavark:container, creating"} interface:

``` {.programlisting .snippet-con}
# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:95:d0:d4 brd ff:ff:ff:ff:ff:ff
    altname enp0s5
    altname ens5
    inet 192.168.124.116/24 brd 192.168.124.255 scope global dynamic noprefixroute eth0
       valid_lft 2290sec preferred_lft 2290sec
    inet6 fe80::bde:e502:5c8d:caf/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

Let\'s run a new `n`{.inlineCode}`ginx`{.inlineCode} container and see
what happens:

``` {.programlisting .snippet-con}
# podman run -d -p 8080:80 \
  --name nginx-netavark
  docker.io/library/nginx
```

When the []{#Chapter_12.xhtml#idx_57c9ba6d .index-entry
index-entry="containers:creating, with Podman"}container
is[]{#Chapter_12.xhtml#idx_30f72776 .index-entry
index-entry="containers:creating, with Netavark"} started, the
`podman0`{.inlineCode} bridge and[]{#Chapter_12.xhtml#idx_e18f2943
.index-entry index-entry="Podman:container, creating"} a
`veth`{.inlineCode} interface []{#Chapter_12.xhtml#idx_8abc7766
.index-entry index-entry="Netavark:container, creating"}appear:

``` {.programlisting .snippet-con}
# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:95:d0:d4 brd ff:ff:ff:ff:ff:ff
    altname enp0s5
    altname ens5
    inet 192.168.124.116/24 brd 192.168.124.255 scope global dynamic noprefixroute eth0
       valid_lft 2225sec preferred_lft 2225sec
    inet6 fe80::bde:e502:5c8d:caf/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: podman0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether b2:4f:dc:12:f3:d1 brd ff:ff:ff:ff:ff:ff
    inet 10.88.0.1/16 brd 10.88.255.255 scope global podman0
       valid_lft forever preferred_lft forever
    inet6 fe80::b04f:dcff:fe12:f3d1/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
4: veth0@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master podman0 state UP group default qlen 1000
    link/ether de:0f:80:78:e7:b6 brd ff:ff:ff:ff:ff:ff link-netns netns-d490e486-f67a-08f4-c066-d87915abadbf
    inet6 fe80::ac62:f2ff:fe0b:94a3/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
```

The interesting part of `veth`{.inlineCode} pairs is that they can be
spawned across multiple network namespaces and that a packet that\'s
sent to one side of the pair is immediately received on the other side.

The `veth0@if2`{.inlineCode} interface[]{#Chapter_12.xhtml#idx_7ca78a3c
.index-entry index-entry="containers:creating, with Netavark"} is linked
to a device that resides in a []{#Chapter_12.xhtml#idx_b366a260
.index-entry index-entry="Podman:container, creating"}network
namespace[]{#Chapter_12.xhtml#idx_00492a4c .index-entry
index-entry="containers:creating, with Podman"} named
`netns-d490e486-f67a-08f4-c066-d87915abadbf`{.inlineCode}.[]{.sentence-end}
Since Linux offers us the option to []{#Chapter_12.xhtml#idx_b26f4a80
.index-entry index-entry="Netavark:container, creating"}inspect network
namespaces using the `ip netns`{.inlineCode} command, we can check
whether the namespace exists and inspect its network stack:

``` {.programlisting .snippet-con}
# ip netns
netns-d490e486-f67a-08f4-c066-d87915abadbf (id: 0)
```

 note
**Hint**

When a new network namespace is created, a file with the same name under
`/var/run/netns/`{.inlineCode} is created.[]{.sentence-end} This file
also has the same inode number that\'s pointed to by the symlink under
`/proc/`{.inlineCode}`<`{.inlineCode}`PID`{.inlineCode}`>`{.inlineCode}`/ns/net`{.inlineCode}.[]{.sentence-end}
When the file is opened, the returned file descriptor gives access to
the namespace.


The preceding command[]{#Chapter_12.xhtml#idx_381940a4 .index-entry
index-entry="containers:creating, with Netavark"} confirms that the
network namespace exists.[]{.sentence-end} Now, we
[]{#Chapter_12.xhtml#idx_ad91b4c8 .index-entry
index-entry="Netavark:container, creating"}want to inspect the network
interfaces that have been defined inside it:

``` {.programlisting .snippet-con}
# ip netns exec netns-d490e486-f67a-08f4-c066-d87915abadbf ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host proto kernel_lo
       valid_lft forever preferred_lft forever
2: eth0@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 0a:86:78:41:11:8d brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.88.0.2/16 brd 10.88.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::886:78ff:fe41:118d/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
```

Previously, we[]{#Chapter_12.xhtml#idx_9dde31f5 .index-entry
index-entry="containers:creating, with Podman"} executed an
`ip addr show`{.inlineCode} command
that\'s[]{#Chapter_12.xhtml#idx_635f0fd6 .index-entry
index-entry="Podman:container, creating"} nested inside the
`ip netns exec`{.inlineCode} command.[]{.sentence-end} The output shows
us an interface that is on the other side of our `veth`{.inlineCode}
pair.[]{.sentence-end} This also tells us something valuable: the
container\'s IPv4 address, set to `10.88.0.2`{.inlineCode}.

 note
**Hint**

When using Podman\'s default network with the `host-local`{.inlineCode}
IPAM plugin, the container IP configuration is persisted to the
`/run/containers/networks/ipam.db`{.inlineCode} file.


We can also inspect the container\'s routing tables:

``` {.programlisting .snippet-con}
# ip netns exec netns-d490e486-f67a-08f4-c066-d87915abadbf ip route show
default via 10.88.0.1 dev eth0 proto static metric 100
10.88.0.0/16 dev eth0 proto kernel scope link src 10.88.0.2
```

All the outbound []{#Chapter_12.xhtml#idx_0bbc4f5f .index-entry
index-entry="containers:creating, with Netavark"}traffic that\'s
directed to the external networks will go through the
`10.88.0.1`{.inlineCode} address, which has been assigned to the
`podman0`{.inlineCode} bridge.

If the container is []{#Chapter_12.xhtml#idx_8fe85d12 .index-entry
index-entry="containers:creating, with Podman"}executed in rootless
mode, the `bridge`{.inlineCode} and `veth`{.inlineCode} pairs
will[]{#Chapter_12.xhtml#idx_d285fdd9 .index-entry
index-entry="Podman:container, creating"} be created in a rootless
network []{#Chapter_12.xhtml#idx_7ff41866 .index-entry
index-entry="Netavark:container, creating"}namespace.

 note
**Important** **note**

The rootless network namespace can be inspected in Podman 4 with the
`podman unshare --rootless-netns`{.inlineCode} command.


Finally, let\'s have a look at the firewall rules that Podman creates
for forwarding.[]{.sentence-end} In a default installation using
`nftables`{.inlineCode}, it is possible to show all firewall rules
created by Podman:

``` {.programlisting .snippet-con}
# nft list table inet netavark
```

Thanks to the previous command, you should be able to see all the
firewall rules created by Podman.

In the next subsection, we will learn how to manage and customize
container networks with the CLI tools that are offered by Podman.

## Managing networks with Podman {#Chapter_12.xhtml#h2_301 .heading-2}

The `podman network`{.inlineCode} command provides
[]{#Chapter_12.xhtml#idx_8ef7cb63 .index-entry
index-entry="Podman:networks, managing"}the necessary tools for managing
container[]{#Chapter_12.xhtml#idx_33e563b0 .index-entry
index-entry="networks:managing, with Podman"} networks.[]{.sentence-end}
The following subcommands are available:

-   `create`{.inlineCode}: Creates a new network
-   `connect`{.inlineCode}: Connects to a given network
-   `disconnect`{.inlineCode}: Disconnects from a network
-   `exists`{.inlineCode}: Checks whether a network exists
-   `inspect`{.inlineCode}: Dumps the configuration of a network
-   `prune`{.inlineCode}: Removes unused networks
-   `reload`{.inlineCode}: Reloads container firewall rules
-   `rm`{.inlineCode}: Removes a given network

In this section, you []{#Chapter_12.xhtml#idx_dacc39fe .index-entry
index-entry="Podman:networks, managing"}will learn how to create a new
network and connect a []{#Chapter_12.xhtml#idx_5ae8f4b6 .index-entry
index-entry="networks:managing, with Podman"}container to
it.[]{.sentence-end} For Podman 4, all the generated Netavark config
files for rootful networks are written to
`/etc/containers/networks`{.inlineCode}, while the config files for
rootless networks are written to
`~/.local/share/containers/storage/networks`{.inlineCode}.

The following command creates a new network called
`example1`{.inlineCode}:

``` {.programlisting .snippet-con}
# podman network create \
  --driver bridge \
  --gateway "10.89.0.1" \
  --subnet "10.89.0.0/16" example1
```

Here, we provided `subnet`{.inlineCode} and `gateway`{.inlineCode}
information, along with the `driver`{.inlineCode} type.[]{.sentence-end}
The resulting network configuration is written in the aforementioned
paths according to the kind of network backend and can be inspected with
the `podman network inspect`{.inlineCode} command.

The following output shows the configuration for the Netavark network
backend:

``` {.programlisting .snippet-con}
# podman network inspect example1
[
     {
          "name": "example1",
          "id": "fec1abe68af4f46969dfb144e5ab0fe49b319a82f3774f232f86f2e2a25d25a8",
          "driver": "bridge",
          "network_interface": "podman1",
          "created": "2025-07-11T18:49:19.994867142Z",
          "subnets": [
               {
                    "subnet": "10.89.0.0/16",
                    "gateway": "10.89.0.1"
               }
          ],
          "ipv6_enabled": false,
          "internal": false,
          "dns_enabled": true,
          "ipam_options": {
               "driver": "host-local"
          },
          "containers": {}
     }
]
```

The new[]{#Chapter_12.xhtml#idx_2d6ec5a5 .index-entry
index-entry="Podman:networks, managing"} network configuration shows
that a bridge called `podman1`{.inlineCode} will
be[]{#Chapter_12.xhtml#idx_c3f29381 .index-entry
index-entry="networks:managing, with Podman"} created for this network
and that containers will allocate IPs from the
`10.89.0.0/16`{.inlineCode} subnet.

The bridge naming convention with Netavark uses the
`podmanN`{.inlineCode} pattern, with *N* *\>=* *0*.

To list all the existing networks, we can use the
`podman network ls`{.inlineCode} command:

``` {.programlisting .snippet-con}
# podman network ls
NETWORK ID    NAME        DRIVER
a8ca04a41ef3  example1    bridge
2f259bab93aa  podman      bridge
```

Now, it\'s time to spin up a container that\'s attached to the new
network.[]{.sentence-end} The following code creates a PostgreSQL
database that\'s attached to the `example1`{.inlineCode} network:

``` {.programlisting .snippet-con}
# podman run -d -p 5432:5432 \
  --network example1 \
  -e POSTGRES_PASSWORD=password \
  --name postgres \
  docker.io/library/postgres
533792e9522fc65371fa6d694526400a3a01f29e6de9b2024e84895f354ed2bb
```

The new container receives an address from the
`10.89.0.0/16`{.inlineCode} subnet, as shown by the
`podman inspect`{.inlineCode} command:

``` {.programlisting .snippet-con}
# podman inspect postgres --format '{{.NetworkSettings.Networks.example1.IPAddress}}'
10.89.0.2
```

With the Netavark[]{#Chapter_12.xhtml#idx_30b8d82e .index-entry
index-entry="Podman:networks, managing"} network backend, we can
double-check this information[]{#Chapter_12.xhtml#idx_5c152065
.index-entry index-entry="networks:managing, with Podman"} by looking at
the contents of the `/run/containers/networks/`{.inlineCode} folder:

``` {.programlisting .snippet-con}
# ls -la /run/containers/networks/
total 28
drwxr-xr-x. 4 root root   120 Jul 11 18:54 .
drwx------. 4 root root    80 Jul 11 18:49 ..
drwxr-xr-x. 2 root root    80 Jul 11 18:54 aardvark-dns
-rw-r--r--. 1 root root     0 Jul 11 18:54 aardvark.lock
drwxr-xr-x. 4 root root   120 Jul 11 18:54 firewall
-rw-------. 1 root root 32768 Jul 11 18:54 ipam.db
```

Looking at the content of the `ipam.db`{.inlineCode} file, we find the
following:

``` {.programlisting .snippet-con}
# cat /run/containers/networks/ipam.db | grep 10.89.0.2
grep: (standard input): binary file matches
```

Podman uses BoltDB, a key-value store, to persist IPAM data that is
saved in that binary file.

We can also see that a new Linux bridge has been created:

``` {.programlisting .snippet-con}
# ip addr show podman1
3: podman1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether c2:74:9e:cb:4b:d4 brd ff:ff:ff:ff:ff:ff
    inet 10.89.0.1/16 brd 10.89.255.255 scope global podman1
       valid_lft forever preferred_lft forever
    inet6 fe80::c074:9eff:fecb:4bd4/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
```

The new device is connected to one peer of the PostgreSQL container\'s
`veth`{.inlineCode} pair:

``` {.programlisting .snippet-con}
# bridge link show
4: veth0@eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master podman1 state forwarding priority 32 cost 2
```

Here, we can see that `veth0@eth0`{.inlineCode} is attached to the
`podman1`{.inlineCode} bridge.[]{.sentence-end} The interface has the
following configuration:

``` {.programlisting .snippet-con}
# ip addr show veth0
4: veth0@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master podman1 state UP group default qlen 1000
    link/ether de:0f:80:78:e7:b6 brd ff:ff:ff:ff:ff:ff link-netns netns-0a6b7e3b-1784-fb72-1598-caba854b137d
    inet6 fe80::dc0f:80ff:fe78:e7b6/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
```

The preceding []{#Chapter_12.xhtml#idx_73aa6c8f .index-entry
index-entry="Podman:networks, managing"}output also shows that the other
side of the `veth`{.inlineCode} pair is[]{#Chapter_12.xhtml#idx_0e2454bf
.index-entry index-entry="networks:managing, with Podman"} located in
the container\'s network namespace---that is,
`netns-0a6b7e3b-1784-fb72-1598-caba854b137d`{.inlineCode}.[]{.sentence-end}
We can inspect the container\'s network configuration and confirm the IP
address that\'s been allocated from the new subnet:

``` {.programlisting .snippet-con}
# ip netns exec netns-0a6b7e3b-1784-fb72-1598-caba854b137d ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host proto kernel_lo
       valid_lft forever preferred_lft forever
2: eth0@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 86:6f:1d:25:fd:68 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.89.0.2/16 brd 10.89.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::846f:1dff:fe25:fd68/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
```

It is possible to []{#Chapter_12.xhtml#idx_1287fef5 .index-entry
index-entry="Podman:networks, managing"}connect a running container to
another network without[]{#Chapter_12.xhtml#idx_689d9df2 .index-entry
index-entry="networks:managing, with Podman"} stopping and restarting
it.[]{.sentence-end} In this way, the container will keep its interface
attached to the original network, and a second interface attached to the
new network is created.[]{.sentence-end} This feature, which is useful
for use cases such as reverse proxies, can be achieved with the
`podman network connect`{.inlineCode} command.[]{.sentence-end} Let\'s
try to run a new `net_example`{.inlineCode} container:

``` {.programlisting .snippet-con}
# podman run -d -p 8080:80 --name net_example docker.io/library/nginx
# podman network connect example1 net_example
```

To verify that the container has been attached to the new network, we
can run the `podman inspect`{.inlineCode} command and look at the
networks:

``` {.programlisting .snippet-con}
# podman inspect net_example
[...omitted output...]
              "Networks": {
                    "example1": {
                         "EndpointID": "",
                         "Gateway": "10.89.0.1",
                         "IPAddress": "10.89.0.3",
                         "IPPrefixLen": 16,
                         "IPv6Gateway": "",
                         "GlobalIPv6Address": "",
                         "GlobalIPv6PrefixLen": 0,
                         "MacAddress": "5a:3b:d9:88:d6:08",
                         "NetworkID": "example1",
                         "DriverOpts": null,
                         "IPAMConfig": null,
                         "Links": null,
                         "Aliases": [
                              "988abb4d2248"
                         ]
                    },
                    "podman": {
                         "EndpointID": "",
                         "Gateway": "10.88.0.1",
                         "IPAddress": "10.88.0.2",
                         "IPPrefixLen": 16,
                         "IPv6Gateway": "",
                         "GlobalIPv6Address": "",
                         "GlobalIPv6PrefixLen": 0,
                         "MacAddress": "16:36:74:8c:87:37",
                         "NetworkID": "podman",
                         "DriverOpts": null,
                         "IPAMConfig": null,
                         "Links": null,
                         "Aliases": [
                              "988abb4d2248"
                         ]
                    }
               }

[…omitted output...]
```

Here, we can see that []{#Chapter_12.xhtml#idx_0703c79e .index-entry
index-entry="Podman:networks, managing"}the container now has two
interfaces attached to []{#Chapter_12.xhtml#idx_2f6f8283 .index-entry
index-entry="networks:managing, with Podman"}the `podman`{.inlineCode}
and `example1`{.inlineCode} networks, with IP addresses allocated from
each network\'s subnet.

To disconnect a container from a network, we can use the
`podman network disconnect`{.inlineCode} command:

``` {.programlisting .snippet-con}
# podman network disconnect example1 net_example
```

When a network is no longer necessary and is disconnected from running
containers, we can delete it with the `podman network rm`{.inlineCode}
command, except for the default network named `podman`{.inlineCode}:

``` {.programlisting .snippet-con}
# podman network rm example1
example1
```

The command\'s output shows the list of removed networks.

 note
**Important** **note**

If the network has associated containers that are either running or have
been stopped, the previous message will fail with **Error: \"example1\"
has associated containers with it**.[]{.sentence-end} To work around
this issue, remove or disconnect the associated containers before using
the command.


The `podman network rm`{.inlineCode} command is
[]{#Chapter_12.xhtml#idx_e6bc7768 .index-entry
index-entry="Podman:networks, managing"}useful when we need to remove a
specific network.[]{.sentence-end} To[]{#Chapter_12.xhtml#idx_b0ef4061
.index-entry index-entry="networks:managing, with Podman"} remove all
unused networks, the `podman network prune`{.inlineCode} command is a
better choice:

``` {.programlisting .snippet-con}
# podman network prune
WARNING! This will remove all networks not used by at least one container.
Are you sure you want to continue? [y/N] y
example2
db_network
```

In this section, we learned about the new Netavark network configuration
manager and how Podman leverages its interface to simplify container
networking.[]{.sentence-end} In a multi-tier or microservices scenario,
we need to let containers communicate with each other.[]{.sentence-end}
In the next section, we will learn how to manage container-to-container
communication.

# Interconnecting two or more containers {#Chapter_12.xhtml#h1_302 .heading-1}

Using our knowledge from the []{#Chapter_12.xhtml#idx_c2810f8a
.index-entry index-entry="containers:interconnecting"}previous section,
we should be aware that two or more containers that have been created
inside the same network can reach each other on the same subnet without
the need for external routing.

At the same time, two or more containers that belong to different
networks will be able to reach each other on different subnets by
routing packets through their networks.

To demonstrate this, let\'s create a couple of `busybox`{.inlineCode}
containers in the same default network:

``` {.programlisting .snippet-con}
# podman run -d --name endpoint1 \
  --cap-add=net_admin,net_raw busybox /bin/sleep 10000
# podman run -d --name endpoint2 \
  --cap-add=net_admin,net_raw busybox /bin/sleep 10000
```

In our lab, the two containers have `10.88.0.3`{.inlineCode}
(`endpoint1`{.inlineCode}) and `10.88.0.4`{.inlineCode}
(`endpoint2`{.inlineCode}) as their addresses.[]{.sentence-end} These
two addresses are subject to change and can be collected using the
methods illustrated previously with the
`podman `{.inlineCode}`inspect`{.inlineCode} or the
`nsenter`{.inlineCode} commands.

Regarding capabilities customization, we added the
`CAP_NET_ADMIN`{.inlineCode} and `CAP_NET_RAW`{.inlineCode} capabilities
to let the containers run commands such as `ping`{.inlineCode} or
`traceroute`{.inlineCode} seamlessly.

Let\'s try to run a `traceroute`{.inlineCode} command from
`endpoint1`{.inlineCode} to `endpoint2`{.inlineCode} to see the path of
a packet:

``` {.programlisting .snippet-con}
# podman exec -it endpoint1 traceroute 10.88.0.4
traceroute to 10.88.0.4 (10.88.0.4), 30 hops max, 46 byte packets
 1  10.88.0.4 (10.88.0.4)  0.009 ms  0.007 ms  0.005 ms
```

As we can see, the packet stays on the internal network and reaches the
node without additional hops.

Now, let\'s create a new network, `net1`{.inlineCode}, and connect a
container called `endpoint3`{.inlineCode} to it:

``` {.programlisting .snippet-con}
# podman network create --driver bridge --gateway "10.90.0.1" --subnet "10.90.0.0/16" --opt isolate=false net1
# podman run -d --name endpoint3 --network=net1 --cap-add=net_admin,net_raw busybox /bin/sleep 10000
```

We are explicitly[]{#Chapter_12.xhtml#idx_789937e5 .index-entry
index-entry="containers:interconnecting"} setting
`--opt isolate=false`{.inlineCode} to disable
isolation.[]{.sentence-end} In Podman 6 and up, by default, containers
in one network will be unable to contact containers in another network,
as isolation will be enforced.

The container in our lab gets an IP address of
`10.90.0.2`{.inlineCode}.[]{.sentence-end} Let\'s see the network path
from `endpoint1`{.inlineCode} to `endpoint3`{.inlineCode}:

``` {.programlisting .snippet-con}
# podman exec -it endpoint1 traceroute 10.90.0.2
traceroute to 10.90.0.2 (10.90.0.2), 30 hops max, 46 byte packets
1  host.containers.internal (10.88.0.1)  0.003 ms  0.001 ms  0.006 ms
2  10.90.0.2 (10.90.0.2)  0.001 ms  0.002 ms  0.002 ms
```

This time, the packet has traversed the `endpoint1`{.inlineCode}
container\'s default gateway (`10.88.0.1`{.inlineCode}) and reached the
`endpoint3`{.inlineCode} container, which is routed from the host to the
associated `net1`{.inlineCode} Linux bridge.

Connectivity across containers in the same host is very easy to manage
and understand.[]{.sentence-end} However, we are still missing an
important aspect for container-to-container communication: DNS
resolution.

Let\'s learn how to leverage this feature with Podman networks.

## Containers DNS resolution {#Chapter_12.xhtml#h2_303 .heading-2}

Despite its many configuration[]{#Chapter_12.xhtml#idx_44805019
.index-entry index-entry="containers DNS resolution"} caveats, DNS
resolution is a very simple concept: a service is queried to provide the
IP address associated with a given hostname.[]{.sentence-end} The amount
of information that can be provided by a DNS server is far richer than
this, but we want to focus on simple IP resolution in this example.

For example, let\'s imagine a scenario where a web application running
on a container named `webapp`{.inlineCode} needs read/write access to a
database running on a second container named
`db`{.inlineCode}.[]{.sentence-end} DNS resolution enables
`webapp`{.inlineCode} to query for the `db`{.inlineCode} container\'s IP
address before contacting it.

Podman\'s default network does not provide DNS resolution, while new
user-created networks have DNS resolution enabled by
default.[]{.sentence-end} On a Netavark network backend, the DNS
resolution is delivered by `aarvark-dns`{.inlineCode}.

To test this feature, we[]{#Chapter_12.xhtml#idx_b13737e1 .index-entry
index-entry="containers DNS resolution"} are going to reuse the
**students** web application that we illustrated in *[Chapter
11](#Chapter_11.xhtml#h1_273){.chapref}*, *Troubleshooting and
Monitoring Containers*, since it provides an adequate client-server
example with a minimal REST service and a database backend based on
PostgreSQL.

 note
**Note**

The source code is available in this book\'s GitHub repository at
[[https://github.com/PacktPublishing/Podman-for-DevOps/tree/main/Chapter11/students]{.url}](https://github.com/PacktPublishing/Podman-for-DevOps/tree/main/Chapter11/students){style="text-decoration: none;"}.


In this example, the web application simply prints some output in JSON
as the result of an HTTP `GET`{.inlineCode} that triggers a query to a
PostgreSQL database.[]{.sentence-end} For our demonstration, we will run
both the database and the web application on the same network.

First, we must create the PostgreSQL database Pod while providing a
generic username and password:

``` {.programlisting .snippet-con}
# podman run -d \
   --network net1 --name db \
   -e POSTGRES_USER=admin \
   -e POSTGRES_PASSWORD=password \
   -e POSTGRES_DB=students \
   postgres
```

Next, we must restore the data from the SQL dump in the
`students`{.inlineCode} folder to the database:

``` {.programlisting .snippet-con}
# cd Chapter11/students
# cat db.sql | podman exec -i db psql -U admin students
```

If you haven\'t already built it in the previous chapters, then you need
to build the `students`{.inlineCode} container image and run it on the
host:

``` {.programlisting .snippet-con}
# buildah build -t students .
# podman run -d \
   --network net1 \
   -p 8080:8080 \
   --name webapp \
   students \
   students -host db -port 5432 \
   -username admin -password password
```

Notice the highlighted part of the command: the `students`{.inlineCode}
application accepts the `-host`{.inlineCode}, `-port`{.inlineCode},
`-username`{.inlineCode}, and `-password`{.inlineCode} options to
customize the database\'s endpoints and credentials.

We did not provide any[]{#Chapter_12.xhtml#idx_9d3e7eb0 .index-entry
index-entry="containers DNS resolution"} IP address in the
`host`{.inlineCode} field.[]{.sentence-end} Instead, the Postgres
container name, `db`{.inlineCode}, along with the default
`5432`{.inlineCode} port, was used to identify the database.

Also, notice that the `db`{.inlineCode} container was created without
any kind of port mapping: we expect to directly reach the database over
the `net1`{.inlineCode} container network, where both containers were
created.

Let\'s try to call the `students`{.inlineCode} application API and see
what happens:

``` {.programlisting .snippet-con}
# curl localhost:8080/students
{"Id":10149,"FirstName":"Frank","MiddleName":"Vincent","LastName":"Zappa","Class":"3A","Course":"Composition"}
```

The query worked fine, meaning that the application successfully queried
the database.[]{.sentence-end} But how did this happen? How did it
resolve the container IP address by only knowing its name? In the next
section, we\'ll look at the behavior of the Netavark network backend.

### DNS resolution on a Netavark network backend {#Chapter_12.xhtml#h3_304 .heading-3}

In the[]{#Chapter_12.xhtml#idx_be211d32 .index-entry
index-entry="Netavark network backend:DNS resolution"} preceding example
with a []{#Chapter_12.xhtml#idx_2febf738 .index-entry
index-entry="DNS resolution:on Netavark network backend"}Netavark
network backend, the `aardvark-dns`{.inlineCode} daemon would be
responsible for container resolution.

The `aardvark-dns`{.inlineCode} project is a companion project of
Netavark written in Rust.[]{.sentence-end} It is a lightweight
authoritative DNS service that can work on both IPv4 A records and IPv6
AAAA records.[]{.sentence-end} It resolves both external and internal
requests.[]{.sentence-end} It allows containers to query other
containers by either name or alias, which can be added during container
creation.[]{.sentence-end} This only works for containers in the same
network as you, though; if you are in `net1`{.inlineCode} and the
`db`{.inlineCode} container is in `net2`{.inlineCode},
`ping db`{.inlineCode} will not work.[]{.sentence-end} Aardvark knows
which networks a container belongs to and only responds with containers
in those networks.

When a new network with DNS resolution enabled is created, a new
`aardvark-dns`{.inlineCode} process is created, as shown in the
following code:

``` {.programlisting .snippet-con}
# ps aux | grep aardvark-dns
root        9115  0.0  0.0 344732  2584 pts/0    Sl   20:15   0:00 /usr/libexec/podman/aardvark-dns --config /run/containers/networks/aardvark-dns -p 53 run
root       10831  0.0  0.0   6400  2044 pts/0    S+   23:36   0:00 grep --color=auto aardvark-dns
```

The process listens on port `53/udp`{.inlineCode} of the host network
namespace for rootful containers and on port `53/udp`{.inlineCode} of
the rootless network namespace for rootless containers.[]{.sentence-end}
This setting can be changed in `containers.conf`{.inlineCode} with the
`dns_bind_port`{.inlineCode} option, so Aardvark can still be run on
systems that already run a DNS server.

The output of the `ps`{.inlineCode} command also shows the default
configuration path---the
`/run/containers/networks/aardvark-dns`{.inlineCode} directory---where
the `aardvark-dns`{.inlineCode} process stores the resolution
configurations under different files, named after the associated
network.[]{.sentence-end} For example, for the `net1`{.inlineCode}
network, we will find content similar to the following:

``` {.programlisting .snippet-con}
# cat /run/containers/networks/aardvark-dns/net1
10.90.0.1
d397b12635f166dd92d56310cf7cc56cac148f3d46c49947d35915f570125767 10.90.0.2  endpoint3,d397b12635f1
1d140d617b04d3697b049195b05b5e37805d9ffbadee66cfabc8df05fc0e07f5 10.90.0.3  db,1d140d617b04
70c0dbfba6f2fd3a451d799d12df16514f6b4e05768f410fec8633093ca4d0bb 10.90.0.6  webapp,70c0dbfba6f2
```

The file stores IPv4[]{#Chapter_12.xhtml#idx_ea255476 .index-entry
index-entry="Netavark network backend:DNS resolution"} addresses (and
IPv6 addresses, if present) for []{#Chapter_12.xhtml#idx_e7b4828e
.index-entry
index-entry="DNS resolution:on Netavark network backend"}every
container.[]{.sentence-end} Here, we can see the containers\' names and
short IDs resolved to the IPv4 addresses.

The first line tells us the address where `aardvark-dns`{.inlineCode} is
listening for incoming requests.[]{.sentence-end} Once again, it
corresponds to the default gateway address for the network.

Let\'s make a test by reaching another container by name:

``` {.programlisting .snippet-con}
$ podman network create web-stack
web-stack

$ podman run -d \
  --name web-service \
  --net web-stack \
  nginx:latest
66b395cc2e4a0e0827c2ef17300b445801c27c9679b3496f6767693f2b310f6a

$ podman run --rm --net web-stack fedora   sh -c "curl -I http://web-service"
...
Server: nginx/1.29.5
Date: Fri, 13 Feb 2026 15:00:40 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Wed, 04 Feb 2026 15:12:20 GMT
Connection: keep-alive
ETag: "698361d4-267"
Accept-Ranges: bytes
...
```

Connecting []{#Chapter_12.xhtml#idx_7b2c931a .index-entry
index-entry="Netavark network backend:DNS resolution"}containers across
the same network allows for fast []{#Chapter_12.xhtml#idx_39278ab9
.index-entry
index-entry="DNS resolution:on Netavark network backend"}and simple
communication across different services running in separate network
namespaces.[]{.sentence-end} However, there are use cases where
containers must share the same network namespace.[]{.sentence-end}
Podman offers a solution to achieve this goal easily: Pods.

## Running containers inside a Pod {#Chapter_12.xhtml#h2_305 .heading-2}

The concept of a Pod comes from the Kubernetes
architecture.[]{.sentence-end} According to the official upstream
documentation, \"*A* *Pod* *is a group of one or more containers, with
shared storage and network resources, and a specification for how to run
the containers*.\"

A Pod is also []{#Chapter_12.xhtml#idx_990b4f5c .index-entry
index-entry="Pods"}the smallest deployable unit in Kubernetes
scheduling.[]{.sentence-end} All the containers inside a Pod share the
same network, UTC, IPC, and (optionally) PID namespace.[]{.sentence-end}
This means that all the services running on the different containers can
refer to each other as *localhost*, while external containers will
continue to contact the pod\'s IP address.[]{.sentence-end} A Pod
receives one IP address that is shared across all the containers.

There are many adoption use cases.[]{.sentence-end} A very common one is
sidecar containers: in this case, a reverse proxy or an OAuth proxy runs
alongside the main container to provide authentication or service mesh
functionalities.

Podman provides the basic tooling for manipulating Pods with the
`podman pod`{.inlineCode} command.[]{.sentence-end} The following
example shows how to create a basic Pod with two containers and
demonstrates network namespace sharing across the two containers in the
Pod.

 note
**Important** **note**

To understand the following example, stop and remove all the running
containers and Pods, and start with a clean environment.


`podman`{.inlineCode}` pod create`{.inlineCode} initializes a new, empty
Pod from scratch:

``` {.programlisting .snippet-con}
# podman pod create --name example_pod
```

 note
**Important** **note**

When a new, empty Pod is created, Podman also creates an
`infra`{.inlineCode} container, which is used to initialize the
namespaces when the Pod is started.[]{.sentence-end} This container is
based on the `k8s.gcr.io/pause`{.inlineCode} image for Podman 3 and a
locally-built `podman-pause`{.inlineCode} image for Podman 4.


Now, we can create two basic `busybox`{.inlineCode} containers inside
the Pod:

``` {.programlisting .snippet-con}
# podman create --name c1 --pod example_pod busybox sh -c 'sleep 10000'
# podman create --name c2 --pod example_pod busybox sh -c 'sleep 10000'
```

Finally, we can start the Pod (and its associated containers) with the
`podman pod start`{.inlineCode} command:

``` {.programlisting .snippet-con}
# podman pod start example_pod
```

Here, we have a running []{#Chapter_12.xhtml#idx_a5706625 .index-entry
index-entry="containers:running, inside Pod"}Pod with two containers
(plus an `infra`{.inlineCode} one) running.[]{.sentence-end}
To[]{#Chapter_12.xhtml#idx_792d1f93 .index-entry
index-entry="Pods:containers, running inside"} verify its status, we can
use the `podman pod ps`{.inlineCode} command:

``` {.programlisting .snippet-con}
# podman pod ps
POD ID        NAME         STATUS      CREATED        INFRA ID      # OF CONTAINERS
8f89f37b8f3b  example_pod  Degraded    8 minutes ago  95589171284a  4
```

With the `podman pod top`{.inlineCode} command, we can see the resources
that are being consumed by each container in the Pod:

``` {.programlisting .snippet-con}
# podman pod top example_pod
USER        PID         PPID        %CPU        ELAPSED          TTY         TIME        COMMAND
root        1           0           0.000       10.576973703s  ?           0s          sleep 1000
0           1           0           0.000       10.577293395s  ?           0s          /catatonit -P
root        1           0           0.000       9.577587032s   ?           0s          sleep 1000
```

After creating the Pod, we can inspect the network\'s
behavior.[]{.sentence-end} First, we will see that only one network
namespace has been created in the system:

``` {.programlisting .snippet-con}
# ip netns
netns-17b9bb67-5ce6-d533-ecf0-9d7f339e6ebd (id: 0)
```

Let\'s check the IP []{#Chapter_12.xhtml#idx_97fe2611 .index-entry
index-entry="containers:running, inside Pod"}configuration for this
namespace and its related network []{#Chapter_12.xhtml#idx_a5c6e04e
.index-entry index-entry="Pods:containers, running inside"}stack:

``` {.programlisting .snippet-con}
# ip netns exec netns-17b9bb67-5ce6-d533-ecf0-9d7f339e6ebd ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0@if15: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether a6:1b:bc:8e:65:1e brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.88.0.3/16 brd 10.88.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::a41b:bcff:fe8e:651e/64 scope link
       valid_lft forever preferred_lft forever
```

To verify that the `c1`{.inlineCode} and `c2`{.inlineCode} containers
share the same network namespace and are running with an IP address of
`10.88.0.3`{.inlineCode}, we can run the same
`ip addr show`{.inlineCode} command inside the containers using the
`podman exec`{.inlineCode} command:

``` {.programlisting .snippet-con}
# podman exec -it c1 ip addr show
# podman exec -it c2 ip addr show
```

These two containers are expected to return the same output as the
`netns-17b9bb67-5ce6-d533-ecf0-9d7f339e6ebd`{.inlineCode} network
namespace.

The example[]{#Chapter_12.xhtml#idx_4e3268c2 .index-entry
index-entry="containers:running, inside Pod"} Pod
[]{#Chapter_12.xhtml#idx_78e29f4b .index-entry
index-entry="Pods:containers, running inside"}can be stopped and removed
with the `podman pod stop`{.inlineCode} and `podman pod rm`{.inlineCode}
commands, respectively:

``` {.programlisting .snippet-con}
# podman pod stop example_pod
# podman pod rm example_pod
```

We will cover Pods in more detail in *[Chapter
14](#Chapter_14.xhtml#h1_330){.chapref}*, *Interacting* *with systemd
and Kubernetes*, where we will also discuss name resolution and
multi-Pod orchestration.

In this section, we focused on communication across two or more
containers inside the same host or Pod, regardless of the number and
type of networks involved.[]{.sentence-end} However, containers are a
platform where you can run services that are generally accessed by the
external world.[]{.sentence-end} For this reason, in the next section,
we will investigate the best practices that can be applied to expose
containers outside their hosts and make their services accessible to
other clients/consumers.

# Exposing containers outside our underlying host {#Chapter_12.xhtml#h1_306 .heading-1}

Container adoption in an enterprise company or a community project could
be a hard thing to do, and could require time.[]{.sentence-end} For this
reason, we may not have all the required services running as containers
during our adoption journey.[]{.sentence-end} This is why exposing
containers outside our underlying host could be a nice solution for
interconnecting services that live in containers to services that run in
the legacy world.

As we briefly saw earlier in this chapter, Podman uses two different
networking stacks, depending on the container: rootless or rootful.

Even though the underlying mechanism is slightly different, depending on
whether you are using a rootless or a rootful container, Podman\'s
command-line options for exposing network ports are the same for both
container types.

 note
**Good to** **know**

Note that the example we are going to see in this section will be
executed as a root user.[]{.sentence-end} This choice was necessary
because the main objective of this section is to show you some of the
firewall configurations that could be mandatory for exposing a container
service to the outside world.


Exposing a container starts with port publishing
activities.[]{.sentence-end} We\'ll learn what this is in the next
section.

## Port publishing {#Chapter_12.xhtml#h2_307 .heading-2}

Port publishing[]{#Chapter_12.xhtml#idx_abd36c46 .index-entry
index-entry="port publishing"} consists of instructing Podman to create
a temporary mapping between the container\'s ports and some random or
custom host\'s ports.

The option to instruct Podman to publish a port is really simple---it
consists of adding the `-p`{.inlineCode} or `--publish`{.inlineCode}
option to the `run`{.inlineCode} command.[]{.sentence-end} Let\'s see
how it works:

``` {.programlisting .snippet-con}
-p=ip:hostPort:containerPort
```

The previous option publishes a container\'s port, or range of ports, to
the host.

You can also specify the protocol:
`-p 127.0.0.1:8080:80/tcp`{.inlineCode}.[]{.sentence-end} Podman
supports forwarding TCP, UDP, and SCTP.[]{.sentence-end} You can do more
than one, but it requires a separate `-p`{.inlineCode} for each.

When we are specifying ranges for `hostPort`{.inlineCode} or
`containerPort`{.inlineCode}, the numbers must be equal for both ranges.

We can even omit `ip`{.inlineCode}.[]{.sentence-end} In that case, the
port will be bound to all the IPs of the underlying
host.[]{.sentence-end} If we do not set the host port, the container\'s
port will be randomly assigned a port on the host.

Let\'s look at an example of the port publishing option:

``` {.programlisting .snippet-con}
# podman run -dt -p 8080:80/tcp docker.io/library/httpd
Trying to pull docker.io/library/httpd:latest...
Getting image source signatures
Copying blob 41c22baa66ec done
Copying blob dcc4698797c8 done
Copying blob d982c879c57e done
Copying blob a2abf6c4d29d done
Copying blob 67283bbdd4a0 done
Copying config dabbfbe0c5 done
Writing manifest to image destination
Storing signatures
ea23dbbeac2ea4cb6d215796e225c0e7c7cf2a979862838ef4299d410c90ad44
```

As you can see, we have []{#Chapter_12.xhtml#idx_c85a77aa .index-entry
index-entry="port publishing"}told Podman to run a container starting
from the `httpd`{.inlineCode} base image.[]{.sentence-end} Then, we
allocated a pseudo-tty (`-t`{.inlineCode}) in detached mode
(`-d`{.inlineCode}) before setting the port mapping to bind the
underlying host port, `8`{.inlineCode}`08`{.inlineCode}`0`{.inlineCode},
to port `80`{.inlineCode} of the container.

Now, we can use the `podman port`{.inlineCode} command to see the actual
mapping:

``` {.programlisting .snippet-con}
# podman ps
CONTAINER ID  IMAGE                           COMMAND           CREATED        STATUS            PORTS               NAMES
ea23dbbeac2e  docker.io/library/httpd:latest  httpd-foreground  3 minutes ago  Up 3 minutes ago  0.0.0.0:8080->80/tcp  ecstatic_chaplygin
# podman port ea23dbbeac2e
80/tcp -> 0.0.0.0:8080
```

First, we requested the list of running containers and then passed the
correct container ID to the `podman port`{.inlineCode}
command.[]{.sentence-end} We can check whether the mapping is working
properly like so:

``` {.programlisting .snippet-con}
# curl localhost:8080
<html><body><h1>It works!</h1></body></html>
```

Here, we executed a `curl`{.inlineCode} command from the host system,
and it worked---the `httpd`{.inlineCode} process running in the
container just replied to us.

If we have multiple ports and we do not care about their assignment on
the underlying host system, we can easily leverage the
`-`{.inlineCode}`P`{.inlineCode} or `--publish-all`{.inlineCode} option
to publish all the ports that are exposed by the
[]{#Chapter_12.xhtml#idx_640b6121 .index-entry
index-entry="port publishing"}container image to random ports on the
host interfaces.[]{.sentence-end} Podman will run through the container
image\'s metadata, looking for the exposed ports.[]{.sentence-end} These
ports are usually defined in a Dockerfile or Containerfile with the
`EXPOSE`{.inlineCode} instruction, as shown here:

``` {.programlisting .snippet-con}
EXPOSE 80/tcp
EXPOSE 80/udp
```

With the previous keyword, we can instruct the container engine that
will run the final container which network ports will be exposed and
used by it.

However, we can leverage an easy but insecure alternative, as shown in
the next section.

## Attaching the host network {#Chapter_12.xhtml#h2_308 .heading-2}

To expose a container service to the []{#Chapter_12.xhtml#idx_a7ccce43
.index-entry index-entry="host network:attaching"}outside world, we can
attach the whole host network to the running container.[]{.sentence-end}
As you can imagine, this method could lead to the unauthorized use of
host resources; for this reason, it is not recommended and should be
used carefully.[]{.sentence-end} Please consider that this gives access
to all ports on the host, including the host\'s localhost, on which many
secure services will open ports.

As we anticipated, attaching the host network to a running container is
quite simple.[]{.sentence-end} Using the right Podman option, we can
easily get rid of any network isolation:

``` {.programlisting .snippet-con}
# podman run --network=host -dt docker.io/library/httpd
deea584a393ea901cf61bf0e3a5490ad0dd5c347c9574bb6de489d7a7a7e0907
```

Here, we used the `--network`{.inlineCode} option while specifying the
`host`{.inlineCode} value.[]{.sentence-end} This informs Podman that we
want to let the container attach to the host network namespace.

After running the previous command, we can check that the running
container is bound to the host system\'s network interfaces since it can
access all of them:

``` {.programlisting .snippet-con}
# netstat -nap|grep ::80
tcp6     0    0 :::80     :::*   LISTEN  37304/httpd
# curl localhost:80
<html><body><h1>It works!</h1></body></html>
```

Here, executing a `curl`{.inlineCode} command from the host confirmed
that the `httpd`{.inlineCode} process inside the container replied
[]{#Chapter_12.xhtml#idx_3fb5d6ff .index-entry
index-entry="host network:attaching"}successfully.

Let\'s also test the opposite case; let\'s connect to the container and
check the network details:

``` {.programlisting .snippet-con}
# podman exec -ti deea584a393e /bin/bash

root@localhost:/usr/local/apache2# apt-get update
...

root@localhost:/usr/local/apache2# apt-get install net-tools
...

root@localhost:/usr/local/apache2# ifconfig -a
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.122.175  netmask 255.255.255.0  broadcast 192.168.122.255
        inet6 fe80::72ec:6b16:6d2a:64c1  prefixlen 64  scopeid 0x20<link>
        ether 52:54:00:66:af:72  txqueuelen 1000  (Ethernet)
        RX packets 67861  bytes 461094320 (439.7 MiB)
        RX errors 0  dropped 188  overruns 0  frame 0
        TX packets 40818  bytes 3603455 (3.4 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
...

root@localhost:/usr/local/apache2# netstat -nap | head -10
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name   
tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN      -                  
tcp        0      0 0.0.0.0:5355            0.0.0.0:*               LISTEN      -                  
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                  
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                  
tcp        0      0 192.168.122.175:57518   151.101.194.132:80      TIME_WAIT   -                  
tcp        0      0 192.168.122.175:57520   151.101.194.132:80      TIME_WAIT   -                  
tcp        0     36 192.168.122.175:22      192.168.122.1:50736     ESTABLISHED -                  
tcp6       0      0 :::5355                 :::*                    LISTEN      -   
```

As you can see, even[]{#Chapter_12.xhtml#idx_94dff9f5 .index-entry
index-entry="host network:attaching"} from the inside, we can see the
host network and any running services and network ports.

The process of exposing containers outside the underlying host does not
stop here.[]{.sentence-end} In the next section, we\'ll learn how to
complete this job.

## Host firewall configuration {#Chapter_12.xhtml#h2_309 .heading-2}

Whether we choose to leverage []{#Chapter_12.xhtml#idx_e71fb5bb
.index-entry index-entry="host firewall configuration"}port publishing
or attach the host network to the container, the process of exposing
containers outside the underlying host does not stop here---we have
reached the base operating system of our host machine.[]{.sentence-end}
In most cases, we will also need to allow the incoming connections to
flow in the host\'s underlying machine, which will be interacting with
the system firewall.

The following example shows a non-comprehensive way to interact with the
base operating system firewall.[]{.sentence-end} If we\'re using a
Fedora operating system or any other Linux distribution that\'s
leveraging `f`{.inlineCode}`irewalld`{.inlineCode} as its firewall
daemon manager, Podman will automatically allow traffic through the
firewall.[]{.sentence-end} No configuration of `firewalld`{.inlineCode}
is required for port forwarding.

 note
**Good to** **know**

`firewalld`{.inlineCode} is a[]{#Chapter_12.xhtml#idx_2f626037
.index-entry index-entry="firewalld"} firewall service daemon that
provides us with an easy and fast way to customize the system
firewall.[]{.sentence-end} `firewalld`{.inlineCode} is dynamic, which
means that it can create, change, and delete the firewall rules without
restarting the firewall daemon each time a change is applied.


As we have seen, the process of exposing the container\'s services is
quite simple but should be performed with a bit of consciousness and
attention: opening a network port to the outside world should always be
done carefully.

# Rootless container network behavior {#Chapter_12.xhtml#h1_310 .heading-1}

As we saw in the previous sections, Podman
[]{#Chapter_12.xhtml#idx_fae9bede .index-entry
index-entry="rootless container network:behavior"}relies on Netavark for
containers running as root and has the privileges to alter network
configurations in the host network namespace.[]{.sentence-end} Until
version 4.3.z, Podman relied solely on the `slirp4netns`{.inlineCode}
project for rootless containers networking.[]{.sentence-end} This
approach allows users to create container network configurations without
the need for root privileges.[]{.sentence-end} With
`slirp4netns`{.inlineCode}, the network interfaces are created inside a
rootless network namespace where the standard user has sufficient
privileges.[]{.sentence-end} This approach allows you to transparently
and flexibly manage rootless container networking.

Podman version 4.4.0
([[https://github.com/containers/podman/blob/main/RELEASE_NOTES.md#440]{.url}](https://github.com/containers/podman/blob/main/RELEASE_NOTES.md#440){style="text-decoration: none;"})
introduced support for a new networking mode based on the
`pasta`{.inlineCode} utility (with lowercase *p*).[]{.sentence-end} This
new networking mode also became the default mode since Podman 5.0.0, and
we will cover it extensively in this chapter.

Let\'s start with some technical background: the
[]{#Chapter_12.xhtml#idx_9c91af90 .index-entry index-entry="pasta"}name
*pasta* is an acronym of **Pack A Subtle Tap Abstraction** and is a
variant []{#Chapter_12.xhtml#idx_02a7e905 .index-entry
index-entry="passt utility:reference link"}of the **passt** utility (an
acronym for **Plug A Simple Socket Transport**).[]{.sentence-end} The
project\'s home page
([[https://passt.top/passt/about/]{.url}](https://passt.top/passt/about/){style="text-decoration: none;"})
provides good insights into their behavior and their motivation.

The `passt`{.inlineCode} utility was created to provide full network
connectivity for virtual machines in user mode without elevated
privileges by providing a translation layer between Layer-2 network
interfaces and Layer-4 sockets on the host.[]{.sentence-end} It supports
QEMU virtualization, and it is a smart and performant way to give the
illusion, from a networking perspective, that application processes
running on the guest virtual machine are running on the local host (that
is, the hypervisor).

The same behavior happens with `pasta`{.inlineCode}, which is a variant
to provide unprivileged connectivity from within a network namespace
and, therefore, a perfect replacement for the `slirp4netns`{.inlineCode}
utility used in previous versions of Podman.[]{.sentence-end} The
general behavior is the same as in `passt`{.inlineCode}, with
`pasta`{.inlineCode} managing the translation from Layer-2 within the
container network namespace to Layer-4 sockets in the host.

Both `pasta`{.inlineCode} and `passt`{.inlineCode} do not need root
privileges and do not need the `CAP_NET_RAW`{.inlineCode} or
`CAP_NET_ADMIN`{.inlineCode} capabilities.

When running rootful containers, we create a `veth`{.inlineCode} pair
between the container network namespace and the host.[]{.sentence-end}
This operation requires `CAP_NET_ADMIN`{.inlineCode} capabilities that
are usually not allowed to unprivileged users for security
reasons.[]{.sentence-end} `pasta`{.inlineCode} creates a tap interface
in the container network namespace and maps process traffic leaving the
container to Layer-4 sockets.[]{.sentence-end} This approach is similar
to the one previously implemented by `slirp4netns`{.inlineCode}, but
doesn\'t require implementing a full TCP/IP stack between the container
and the host for traffic translation, which can also impact
performance.[]{.sentence-end} Please consider also that both
`slirp4netns`{.inlineCode} and `pasta`{.inlineCode} are slower than root
networking with `veth`{.inlineCode} pairs, but `pasta`{.inlineCode} is
much faster than `slirp4netns`{.inlineCode} and almost as fast as root
in some cases.

For every new rootless container, a new `pasta`{.inlineCode} process is
executed on the host.[]{.sentence-end} The process creates a network
namespace for the container under
`/run/user/`{.inlineCode}`<`{.inlineCode}`user`{.inlineCode}`_ID`{.inlineCode}`>`{.inlineCode}`/netns`{.inlineCode},
and all IPv4 and IPv6 addresses and routes are copied from the
host.[]{.sentence-end} The `tap`{.inlineCode} interface created inside
the container inherits the same []{#Chapter_12.xhtml#idx_a0c4393b
.index-entry index-entry="rootless container network:behavior"}name as
the host primary interface, and also port forwarding preserves the
original source IP address.

Such a configuration in `pasta`{.inlineCode} is different from the
previous behavior of `slirp4netns`{.inlineCode}, where a
`tap0`{.inlineCode} device was created with the
`10.0.2.100/24`{.inlineCode} address (from the default
`slirp4netns 10.0.2.0/24`{.inlineCode} subnet).[]{.sentence-end} It also
avoids any[]{#Chapter_12.xhtml#idx_5459d4b4 .index-entry
index-entry="network address translation (NAT)"} kind of **network**
**address** **translation** (**NAT**), which improves performance.

The following example demonstrates the network behavior of a rootless
`busybox`{.inlineCode} container:

``` {.programlisting .snippet-con}
$ podman run -i busybox sh -c 'ip addr show'
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 65520 qdisc fq_codel qlen 1000
    link/ether 9a:36:ed:16:4d:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.94/24 brd 192.168.122.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::9836:edff:fe16:4d11/64 scope link tentative
       valid_lft forever preferred_lft forever
```

To demonstrate that the []{#Chapter_12.xhtml#idx_3294fc48 .index-entry
index-entry="rootless container network:behavior"}container inherits the
same IP address and interface name of the host, let\'s inspect the
host\'s network configuration:

``` {.programlisting .snippet-con}
$ ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:69:94:4d brd ff:ff:ff:ff:ff:ff
    altname enp1s0
    inet 192.168.122.94/24 brd 192.168.122.255 scope global dynamic noprefixroute eth0
       valid_lft 2786sec preferred_lft 2786sec
    inet6 fe80::e93b:f1db:1357:84fa/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

A common misconception is that rootless containers lack independent IP
addresses.[]{.sentence-end} In reality, rootless containers do have
unique IPs, but they exist within a private network namespace managed by
the rootless user.[]{.sentence-end} This creates a secure *bubble* where
containers can talk to one another, but the host machine cannot directly
ping or access those internal IPs.

To enable communication between your rootless services, you generally
have three primary strategies:

-   The simplest way to connect containers is to group them into a
    Pod.[]{.sentence-end} Containers within the same Pod share the same
    network namespace, meaning they share a single IP address and can
    communicate with each other over the localhost interface.
-   When you attach containers to a custom bridge network, Podman
    manages their interfaces within the rootless network
    namespace.[]{.sentence-end} In this environment, each container
    receives its own IP address and can resolve other containers by
    their name.
-   If you need the host or external users to access a containerized
    service, you must use port mapping (the `-p`{.inlineCode}
    flag).[]{.sentence-end} This creates a bridge between a port on the
    host (for example, `8080`{.inlineCode}) and a port inside the
    container\'s private namespace (for example, `80`{.inlineCode}).

Using a Podman 4 []{#Chapter_12.xhtml#idx_53685500 .index-entry
index-entry="rootless container network:behavior"}network backend,
let\'s quickly focus on the second scenario, where two Pods are attached
to a rootless network.[]{.sentence-end} First, we need to create the
network and attach a couple of test containers:

``` {.programlisting .snippet-con}
$ podman network create rootless-net
$ podman run -d --net rootless-net --name endpoint1 --cap-add=net_admin,net_raw busybox /bin/sleep 10000
$ podman run -d --net rootless-net --name endpoint2 --cap-add=net_admin,net_raw busybox /bin/sleep 10000
```

Let\'s try to ping the `endpoint2`{.inlineCode} container from
`endpoint1`{.inlineCode}:

``` {.programlisting .snippet-con}
$ podman exec -it endpoint1 ping -c1 endpoint2
PING endpoint1 (10.89.2.5): 56 data bytes
64 bytes from 10.89.2.5: seq=0 ttl=64 time=0.023 ms
--- endpoint1 ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 0.023/0.023/0.023 ms
```

These two containers can communicate on the common network and have
different IPv4 addresses.[]{.sentence-end} To prove this, we can inspect
the contents of the `aardvark-dns`{.inlineCode} configuration for the
rootless containers:

``` {.programlisting .snippet-con}
$ cat /run/user/1000/containers/networks/aardvark-dns/rootless-net
10.89.2.1
fe27f8d653384fc191d5c580d18d874d480a7e8ef74c2626ae21b118eedbf1e6 10.89.2.4  endpoint1,fe27f8d65338
19a4307516ce1ece32ce58753e70da5e5abf9cf70feea7b981917ae399ef934d 10.89.2.5  endpoint2,19a4307516ce
```

To prove that rootless networking still follows standard Linux
networking principles, we can peek inside the private network namespace
that Podman manages for your user.[]{.sentence-end} This namespace
contains the virtual `bridge`{.inlineCode} and `veth`{.inlineCode} pairs
that allow your containers to talk to one another.

It is important to []{#Chapter_12.xhtml#idx_57e6f581 .index-entry
index-entry="rootless container network:behavior"}understand that these
interfaces are entirely private to your user session.[]{.sentence-end}
Unlike rootful Podman, you will not see these `veth`{.inlineCode}
interfaces if you run `ip link`{.inlineCode} on your host
machine.[]{.sentence-end} Because a non-privileged user cannot modify
the host\'s networking stack, these pairs exist only within the isolated
rootless namespace, safely tucked away from the rest of the system.

To see these *invisible* interfaces, we must use the
`podman unshare`{.inlineCode} command to enter the rootless network
namespace:

``` {.programlisting .snippet-con}
$ podman unshare --rootless-netns ip link | grep 'podman'
3: podman3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
4: veth0@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master podman3 state UP mode DEFAULT group default qlen 1000
5: veth1@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master podman3 state UP mode DEFAULT group default qlen 1000
```

Another limitation of rootless containers is the `ping`{.inlineCode}
command.[]{.sentence-end} Usually, on Linux distributions, standard
non-root users lack the `CAP_NET_RAW`{.inlineCode} security
capability.[]{.sentence-end} This inhibits the execution of the
`ping`{.inlineCode} command, which leverages the send/receive operations
of ICMP packets.[]{.sentence-end} If we want to use the
`ping`{.inlineCode} command in a rootless container, we can enable the
missing security capability through the `sysctl`{.inlineCode} command:

``` {.programlisting .snippet-con}
# sysctl -w "net.ipv4.ping_group_range=0 2000000"
```

Note that this could allow any process that will be executed by a user
on these groups to send `ping`{.inlineCode} packets.

Users running rootless []{#Chapter_12.xhtml#idx_d5cf0772 .index-entry
index-entry="rootless container network:behavior"}containers executed
with `pasta`{.inlineCode} could find it necessary to work in legacy mode
with the same network configuration of
`slirp4netns`{.inlineCode}.[]{.sentence-end} For those use cases, it is
possible to run a container with the following backward-compatible
configuration:

``` {.programlisting .snippet-con}
$ podman run \     --network=pasta:--ipv4-only,-a,10.0.2.0,-n,24,-g,10.0.2.2,--dns-forward,10.0.2.3,-m,1500,--no-ndp,--no-dhcpv6,--no-dhcp --rm -d busybox sleep 1000
```

By using `pasta`{.inlineCode}, users can customize the behavior of the
container by tweaking various parameters.[]{.sentence-end} For example,
it is possible to retain the default configuration while passing a
custom MTU on the container interface, as in the following example:

``` {.programlisting .snippet-con}
$ podman run --network=pasta:-m,1500 \
--rm -d busybox sleep 1000
```

Also, port mapping works in a similar way to rootful
containers.[]{.sentence-end} The biggest change is that we do not have
permission to update the firewall, so it may be necessary to add
firewall rules to allow traffic through.[]{.sentence-end} The following
example runs a minimal Python HTTP server and binds port
`8000`{.inlineCode} in the container to port `8000`{.inlineCode} on the
host:

``` {.programlisting .snippet-con}
$ podman run -d --rm -p 8000:8000 \                    registry.access.redhat.com/ubi9/python-312 \
python -m http.server
```

This is how the `pasta`{.inlineCode} process created with the container
maps the port:

``` {.programlisting .snippet-con}
$ ps aux | grep pasta
packt  189281  0.0  0.0 206444 17132 ?        Ss   00:18   0:00 /usr/bin/pasta --config-net -t 8000-8000:8000-8000 --dns-forward 169.254.1.1 -u none -T none -U none --no-map-gw --quiet --netns /run/user/1000/netns/netns-fa625228-da6d-e87d-d62f-bee84d158d8a --map-guest-addr 169.254.1.2
packt  189580  0.0  0.0 231248  2380 pts/3    S+   00:19   0:00 grep --color=auto pasta
```

The `-t`{.inlineCode} option[]{#Chapter_12.xhtml#idx_3b73830f
.index-entry index-entry="rootless container network:behavior"}
configures the TCP port forwarding to the guest (in our case, the
container).

Finally, while using rootless containers, we also need to consider that
the port publishing technique can only be used for ports above
`1024`{.inlineCode}.[]{.sentence-end} This is because, on Linux
operating systems, all the ports below `1024`{.inlineCode} are
privileged and cannot be used by standard non-root users.

# Summary {#Chapter_12.xhtml#h1_311 .heading-1}

In this chapter, we learned how container network isolation can be
leveraged to allow network segregation for each container that\'s
running through network namespaces.[]{.sentence-end} These activities
seem complex, but thankfully, with the help of a container runtime, the
steps are almost automated.[]{.sentence-end} We learned how to manage
container networking with Podman and how to interconnect two or more
containers.[]{.sentence-end} Finally, we learned how to expose a
container\'s network ports outside of the underlying host and what kind
of limitations we can expect while networking for rootless containers.

In the next chapter, we will discover the main differences between
Docker and Podman.[]{.sentence-end} This will be useful for advanced
users, but also for novice ones, to understand what we can expect by
comparing these two container engines.

# Further reading {#Chapter_12.xhtml#h1_312 .heading-1}

To learn more about the topics that were covered in this chapter, take a
look at the following resources:

-   The Netavark project on GitHub:
    [[https://github.com/containers/netavark]{.url}](https://github.com/containers/netavark){style="text-decoration: none;"}
-   The `a`{.inlineCode}`ardvark-dns`{.inlineCode} project on GitHub:
    [[https://github.com/containers/aardvark-dns]{.url}](https://github.com/containers/aardvark-dns){style="text-decoration: none;"}
-   Kubernetes Pod definition:
    [[https://kubernetes.io/docs/concepts/workloads/pods/]{.url}](https://kubernetes.io/docs/concepts/workloads/pods/){style="text-decoration: none;"}
-   The `passt`{.inlineCode}/`pasta`{.inlineCode} project repository:
    [[https://passt.top/passt/about/]{.url}](https://passt.top/passt/about/){style="text-decoration: none;"}

# Join us on Discord {#Chapter_12.xhtml#h1_313 .heading-1}

For discussions around the book and to connect with your peers, join us
on Discord at
[[packt.link/discordcloud]{.url}](https://packt.link/discordcloud){style="text-decoration: none;"}
or scan the QR code below:

![Image](images/B31467_12_1.png){style="width:25%;"}


[]{#Chapter_13.xhtml}

 {.section .chapter-first}