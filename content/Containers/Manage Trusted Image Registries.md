---
title: Managing Trusted Image Registries 
draft: false
summary: Managing Trusted Image Registires and creating Aliases
---

`/etc/containers/registries.conf` 
- Overridden by the user-related `$HOME/.config/containers/registries.conf` file if present
- Manages a list of trusted registries that Podman can safely contact to search and pull images.

Let\'s look at an example of this file:

```
unqualified-search-registries = ["docker.io", "quay.io"]

[[registry]]
location = "registry.example.com:5000" 
insecure = false 
```

- Podman accepts both **unqualified** and **fully** **qualified** images. 

**fully qualified**
- Includes a registry server FQDN, namespace, image name, and tag.
- i.e. `docker.io/library/nginx:latest` 
	- Has a full name that cannot be confused with any other Nginx image.

**unqualified image** 
- Only includes the image's name. 
- i.e. `nginx` image can have multiple instances in the searched registries. 
- The majority of the images that result from the basic `podman search nginx` command will not be official and should be analyzed in detail to ensure they're trusted. 
	- Output can be filtered by the `OFFICIAL` flag and by the number of stars 

## Registry configuration file global settings

`unqualified-search-registry`
- Defines the search list of registries for unqualified images. 
- When the user runs the `podman search <image_name>` command, Podman will search across the registries defined in this list.
- By removing a registry from the list, Podman will stop searching the registry. 
- Podman will still be able to pull a fully qualified image from a foreign registry.

## Registries and mirrors

`[[registry]]`
- Manage single registries and create matching patterns for specific images.
- The main settings of these tables:
	- `prefix`
		- Used to define the image names and can support multiple formats. 
		- Can define images by following the `host[:port]/namespace[/_namespace_…]/repo(:_tag|@digest)` pattern.
		- Simpler patterns, such as `host[:port]`, `host[:port]/namespace`, and even `[*.]host`, can be applied. 
		- Users can define a generic prefix for a registry or a more detailed prefix to match a specific image or tag. 
		- Given a fully qualified image, if two `[[registry]]` tables have a prefix with a partial match, the longest matching pattern will be used.
	- `insecure`
		- Boolean value
		- Allows unencrypted HTTP connections or TLS connections based on untrusted certificates.
	- `blocked`
		- This is a Boolean (`true` or `false`) that\'s used to define blocked registries. If it\'s set to `true`, the registries or images that match the prefix are blocked.
	- `location`
		- Defines the registry's location. 
		- By default, it is equal to `prefix`
		- A pattern that matches a custom prefix namespace will resolve to the `location` value.

`[[registry.mirror]]` 
- Provide alternate paths to the main registry or registry namespace.
- When multiple mirrors are provided, Podman will search across them first and then fall back to the location that's defined in the main `[[registry]]` table.

The following example extends the previous one by defining a namespaced registry entry and its mirror:

```
unqualified-search-registries = ["docker.io", "quay.io"]

[[registry]] 
location = "registry.example.com:5000/foo" 
insecure = false 

[[registry.mirror]] 
location = "mirror1.example.com:5000/bar" 

[[registry.mirror]] 
location = "mirror2.example.com:5000/bar" 
```

According to this example, if a user tries to pull the image tagged as `registry.example.com:5000/foo/app:latest`, Podman will try `mirror1.example.com:5000/bar/app:latest`, then `mirror2.example.com:5000/bar/app:latest`, and fall back to `registry.example.com:5000/foo/app:latest` in case a failure occurs.

Using a prefix provides even more flexibility. In the following example, all the images that match `example.com/foo` will be redirected to mirror locations and fall back to the main location at the end:

```
unqualified-search-registries = ["docker.io", "quay.io"] 

[[registry]] 
prefix = "example.com/foo" 
location = "registry.example.com:5000/foo" 
insecure = false 

[[registry.mirror]] 
location = "mirror1.example.com:5000/bar" 

[[registry.mirror]] 
location = "mirror2.example.com:5000/bar" 
```

In this example, when we pull the `example.com/foo/app:latest` image, Podman will attempt `mirror1.example.com:5000/bar/app:latest`, followed by `mirror2.example.com:5000/bar/app:latest` and `registry.example.com:5000/foo/app:latest`.

- Possible to use mirroring for replacing public registries with private mirrors in disconnected environments. 

The following example remaps the `docker.io` and `quay.io` registries to a private mirror with different namespaces:

```
[[registry]] 
prefix="quay.io" 
location="mirror-internal.example.com/quay" 

[[registry]] 
prefix="docker.io" 
location="mirror-internal.example.com/docker" 
```

- Mirror registries should be kept up to date with mirrored repositories.
- Administrators or SRE teams should implement an image sync policy to keep the repositories updated.

- Can block a source that is not considered trusted. 
	- Could impact a single image, a namespace, or a whole registry.

The following example tells Podman not to search for or pull images from a blocked registry:  
```
[[registry]] 
location = "registry.rogue.io" 
blocked = true 
```

- Can refine the blocking policy by passing a specific namespace without blocking the whole registry. 

In the following example, every image search or pull that matches the `quay.io/foo` namespace pattern defined in the `prefix` field is blocked:
```
[[registry]] 
prefix = "quay.io/foo/" 
location = "quay.io" 
blocked = true 
```

- Can define a very specific blocking rule for a single image or even a single tag by using the same approach that was described for namespaces. 

Block a specific image tag:

```
[[registry]]
prefix = "internal-registry.example.com/dev/app:v0.1" 
location = "internal-registry.example.com " 
blocked = true 
```

- It is possible to combine many blocking rules and add mirror tables on top of them.

- A clever idea would be to keep the registry's configuration file updated using configuration management tools and declaratively apply the registry's filters.

## Aliases
- **Fully qualified image names** (**FQNs**) can become quite long if we sum up the registry FQDN, namespace(s), repository, and tags.
- While using FQNs such as `quay.io/podman/stable:latest` is the gold standard for security and automation, they are often cumbersome to type manually. 
- To bridge the gap between convenience and security, Podman utilizes a short-name aliasing system.
- Normally, when you provide an *unqualified* name (for example, `podman pull fedora`), Podman must iterate through a list of registries defined in the `[registries.search]` table of your configuration. 
- This can be slow and, if not configured correctly, potentially insecure.
- Aliases change this behavior by short-circuiting the search. 
- If a short name matches an entry in an alias table, Podman immediately resolves it to the specific, fully qualified name provided in the mapping, bypassing the search registry list entirely.
- On Fedora and RHEL, you don\'t have to build this list from scratch. 
- The system comes pre-configured with an extensive library of trusted mappings located at the following:
```
/etc/containers/registries.conf.d/000-shortnames.conf 
```
- This file is maintained by the community and operating system vendors to ensure that common names point to their most logical and secure upstream sources. 
- For example, an entry in this file might look as follows:
```
[aliases] 
"fedora" = "registry.fedoraproject.org/fedora" 
"ubi8" = "registry.access.redhat.com/ubi8" 
"alpine" = "docker.io/library/alpine" 
```
- When an alias matches a short name, it is immediately used without the registries defined in the `unqualified-search-registries` list being searched.
- Can create custom files inside the `/etc/containers/registries.conf.d/` folder to define aliases without bloating the main configuration file.
- No tags or digests: 
	- Aliases resolve the repository path, not a specific version. 
	- For example, if you alias `my-app` to `quay.io/org/my-app`, running `podman pull my-app:v2` will correctly resolve to `quay.io/org/my-app:v2`. 
	- The alias handles the prefix, while the tag remains dynamic.
- Precedence: 
	- Local user-defined aliases (often found in `$HOME/.config/containers/registries.conf.d/`) will typically override the global system aliases found in `/etc/containers/`.