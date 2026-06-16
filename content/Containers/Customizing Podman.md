---
title: Customizing Podman's Behavior
summary: Custom options for Podman configuration
---

Podman's default configuration works out of the box for most use cases, but its configuration is highly flexible. The following configuration files are available for customizing its behavior:

`containers.conf`
- TOML-formatted file 
- Podman runtime configurations
- search paths for `conmon` and container runtime binaries. 
- installed by default under`/usr/share/containers/`  
- Can be overridden by the  `/etc/containers/containers.conf` and `$HOME/.config/containers/containers.conf` files for system-wide and user wide settings, respectively.
- Used to customize the behavior of the engine. 
- Users can influence how the container is created and its life cycle by customizing settings such as logging, DNS resolution, environment variables, shared memory usage, cgroup management, and many others.

`man containers.conf`

`storage.conf`
- TOML-formatted file 
- customize the storage settings that are used by the container engine. 
-  customize the default storage driver, as well as the read/write directory of the container storage (also known as the graph root), which is an additional driver storage option. 
- By default, the driver is set to **overlay**.
- Default path of this file is `/usr/share/containers/storage.conf`
- Overrides can be found or created under `/etc/containers/storage.conf` for system-wide customizations.
- User-scoped configurations that impact rootless containers can be found under `$XDG_CONFIG_HOME/containers/storage.conf` or `$HOME/.config/containers/storage.conf`.

`mounts.conf`
- Defines the volume mounts that should be automatically mounted inside a container when it is started. 
- Useful, for example, to automatically pass secrets such as keys and certificates inside a container.
- Found under `/usr/share/containers/mounts.conf` and overridden by a file located at  `/etc/containers/mounts.conf`.
- In rootless mode, the override file can be placed under `$HOME/.``config/containers/mounts.conf`.

`seccomp.json` 
- JSON file 
- lets users customize the allowed `syscalls` that a process inside a container can perform and define the blocked ones at the same time. 
- The default path for this file is `/usr/share/containers/seccomp.json`.

`man seccomp`

`policy.json`
- JSON file
- Defines how Podman will perform signature verification. 
- Default path of this file is `/etc/containers/policy.json` and can be overridden by the user-scoped `$HOME/.config/containers/policy.json`.
- Accepts three kinds of policies:
    -   `insecureAcceptAnything`: Accept any image from the specified registry
    -   `reject`: Reject any image from the specified registry
    -   `signedBy`: Accept only images signed by a specific, known entity

- The default configuration is to accept every image (the `insecureAcceptAnything` policy)
- can be modified to pull only trusted images that can be verified by a signature. 
- Can define custom GPG keys to verify the signatures and the identity that signed them. 

`man containers-policy.json`

## Customizing the container registries search list 

Podman searches for and downloads images from a list of trusted container registries. The `/etc/containers/registries.conf`
file is a TOML config file that can be used to customize whitelisted registries that are allowed to be searched and used as image sources, as well as registry mirroring and insecure registries without TLS termination. In this config file, the `unqualified-search-registries` key is populated with an array of unqualified registries with no specification regarding image repositories and tags.

On a Fedora system, with a new installation of Podman, this key has the following content:
```
unqualified-search-registries = ["registry.fedoraproject.org", "registry.access.redhat.com", "docker.io", "quay.io"]
```

Users can add or remove registries from this array to let Podman search and pull from them.

Be very cautious when adding registries and use only trusted registries to avoid pulling images containing malicious code.

Private registries usually require additional authentication to be accessed. This can be accomplished with the
`podman login` command.

If the `$HOME/.config/containers/registries.conf` file is found in the user's home directory, it overrides the
`/etc/containers/registries.conf` file. In this way, different users on the same system will be able to run Podman with their custom registry whitelists and mirrors.