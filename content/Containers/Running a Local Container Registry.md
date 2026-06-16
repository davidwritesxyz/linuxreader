---
draft: true
---
- Most companies and organizations adopt enterprise-grade registries to rely on secure and resilient solutions for their container image storage. 
- Most enterprise registries also offer advanced features such as **role-based access control** (**RBAC**), an image vulnerability scanner, mirroring, geo-replication, and high availability, becoming the default choice for production and mission-critical environments.

## Local registries
- Sometimes it is very useful to run a simple local registry
- Useful in development environments or training labs. 
- Local registries can also be helpful in disconnected environments to mirror the main public or private registries.

## Running a containerized registry 


- Must define a destination directory to host all image layers and metadata. 
- `5000/tcp` is default port for this service. 

- Can use Podman or its companion tools, Buildah or Skopeo to push to a registry. 
- With Skopeo,  we do not even need to scope the image name with the registry name.

Push the new image to the registry with Skopeo:
```bash
$ skopeo copy --dest-tls-verify=false \
containers-storage:localhost/minimal_httpd \
docker://localhost:5000/minimal_httpd 
```

`--dest-tls-verify=false`
- Necessary since the local registry doesn't have TLS or a trusted certificate; it provides an HTTP transport by default.

## Customizing the registry configuration 

The registry configuration file (`/etc/docker/registry/config.yml`) can be modified to alter its behavior. The default content of this file is the following:

```yml
version: 0.1

log:
  fields:
    service: registry

storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry

http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]

health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```

Default config has no:
- authentication
- allowed deletion of images
- no TLS encryption. 

The following file contains a modified version of the `config.yml` registry (`Chapter09/local_registry/customizations/config.yml`):

```yml
version: 0.1

log:
  fields:
    service: registry

storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
  delete:
    enabled: true

auth:
  htpasswd:
    realm: basic-realm
    path: /var/lib/htpasswd

http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
  tls:
    certificate: /etc/pki/certs/tls.crt
    key: /etc/pki/certs/tls.key

health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3

```

The highlighted sections in the previous example emphasize the following added features:

- **Image deletion**: By default, this setting is disabled. We need to explicitly enable it.
- **Basic authentication** **using an** `htpasswd` **file**: This approach is suitable for development and lab environments, while a token-based authentication relying on an external issuer would best fit in production use cases.
- **HTTPS transport** **using self-signed certificates**: The `http` section configures the registry\'s HTTP server, including its listening address, custom headers, and TLS settings.

Before running the registry again with our custom configuration, we need to generate an `htpasswd` file that holds at least one valid login and the self-signed certificates for TLS encryption. Let\'s start with the `htpasswd` file -- we can generate it using the `htpasswd` utility, as in the following example:

```
htpasswd -cBb ./htpasswd admin p0dman4Dev0ps#
```

The `-cBb` option enables batch mode (useful to provide the password non-interactively), creates the file if it does not exist, and enables the `bcrypt` hashing function *\[2\]*. In this example, we create the user `admin` with the password `p0dman4Dev0ps#`.

Finally, we need to create a self-signed server certificate with its related private key, to be used for HTTPS connections. As an example, a certificate associated with the `localhost` **Common Name** (**CN**) will be created.

 packt_tip **Important** **note**

Bounding certificates to the `localhost` CN is a frequent practice in development environments. However, if the registry is meant to be exposed externally, the `CN` and `SubjectAltName` fields should map to the host FQDN and alternate names.

The following example shows how to create a self-signed certificate with the `openssl` utility:

```bash
$ mkdir certs 
$ openssl req -newkey rsa:4096 -x509 -sha256 -nodes \
	-days 365 \
	-out certs/tls.crt \
	-keyout certs/tls.key \
	-subj '/CN=localhost' \
	-addext "subjectAltName=DNS:localhost" 
```

The command will issue non-interactive certificate generation, without any extra information about the certificate subject. The `tls.key` private key is generated using a 4,096-bit RSA algorithm. The certificate, named `tls.crt`, is set to expire after 1 year. Both the key and certificate are written inside the `certs` directory.

Inspect the content of the generated certificate:  
``` $ openssl x509 -in certs/tls.crt -text -noout ```


 packt_tip **Hint**

For the purpose of this example, the self-signed certificate is acceptable, but it should be avoided in production scenarios.

Solutions such as **Let\'s Encrypt** provide a free CA service for everybody and can be used to reliably secure the registry or any other HTTPS service. For further details, visit https://letsencrypt.org/.

We now have all the requirements to run our custom registry. Before creating the new container, make sure the previous instance has been stopped and removed:

```
# podman stop local_registry && podman rm local_registry 
```

The next command shows how to run the new custom registry using bind mounts to pass the certificates folder, the `htpasswd` file, the registry store, and, obviously, the custom config file:

```
# podman volume create registry_data
# podman run -d --name local_registry \
-p 5000:5000 \
-v $PWD/htpasswd:/var/lib/htpasswd:z \
-v $PWD/config.yml:/etc/docker/registry/config.yml:z \
-v registry_data:/var/lib/registry:Z \
-v $PWD/certs:/etc/pki/certs:z \
--restart=always \
registry:2 
```

We can now test the login to the remote registry using the previously defined credentials:

``` $ skopeo login -u admin -p p0dman4Dev0ps# --tls-verify=false localhost:5000 Login Succeeded! ```

Notice the `--tls-verify=false` option to skip TLS certificate validation. Since it is a self-signed certificate, we need to bypass checks that would produce the `x509: certificate signed by unknown authority` error message.

We can try again to delete the image pushed before:

``` $ skopeo delete \ --tls-verify=false \ docker://localhost:5000/minimal_httpd ```

This time, the command will succeed since the deletion feature was enabled in the config file.

A local registry can be used to mirror images from an external public registry. In the next subsection, we will see an example of registry mirroring using our local registry and a selected set of repositories and images.

## Using a local registry to sync repositories 

Mirroring images and repositories to a local registry can be very useful in disconnected environments. This can also be very useful to keep an `async` copy of selected images and be able to keep pulling them during public service outages.

The next example shows simple mirroring using the `skopeo sync` command with a list of images provided by a YAML file and our local registry as the destination:

``` $ skopeo sync \ --src yaml --dest docker \ --dest-tls-verify=false \ kube_sync.yaml localhost:5000 ```

The YAML file contains a list of the images that compose a Kubernetes control plane for a specific release. Again, we take advantage of regular expressions to customize the images to pull (`Chapter09/kube_sync.yaml`):

``` {.programlisting .snippet-code} k8s.gcr.io: tls-verify: true images-by-tag-regex: kube-apiserver: ^v1\.3 2\..* kube-controller-manager: ^v1\.3 2\..* kube-proxy: ^v1\.3 2\..* kube-scheduler: ^v1\.3 2\..* coredns/coredns: ^v1\.9 \..* etcd: 3\.5 .[0-9]*-[0-9]* ```

When synchronizing a remote and local registry, a lot of layers can be mirrored in the process. For this reason, it is important to monitor the storage used by the registry (`/var/lib/registry` in our example) to avoid filling up the filesystem.

When the filesystem is filled, deleting older and unused images with Skopeo is not enough, and an extra garbage collection action is necessary to free space. The next subsection illustrates this process.

## Managing registry garbage collection 

When a `delete` command is issued on a container registry, it only deletes the image manifests that reference a set of blobs (which could be layers or further manifests), while keeping the blobs in the filesystem.

If a blob is no longer referenced by any manifest, it can be eligible for garbage collection by the registry. The garbage collection process is managed with a dedicated command, `registry garbage-collect`, issued inside the registry container. This is not an automatic process and should be executed manually or scheduled.

In the next example, we will run a simple garbage collection. The `--dry-run` flag only prints the eligible blobs that are no longer referenced by a manifest, and thus they can be safely deleted:

```
# podman exec -it local_registry \ registry garbage-collect --dry-run \ /etc/docker/registry/config.yml 
```

To delete the blobs, simply remove the `--dry-run` option:

```
# podman exec -it local_registry \ registry garbage-collect /etc/docker/registry/config.yml 
```

Garbage collection helps keep the registry free of unused blobs and saves storage space. On the other hand, we must keep in mind that an unreferenced blob could still be reused in the future by another image. If deleted, it could be necessary to upload it again eventually.
