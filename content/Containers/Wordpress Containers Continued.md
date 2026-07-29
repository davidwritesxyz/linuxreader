---
title: Wordpress Containers Continued
draft: true
---
## Converting compose.yml to quadlets


Set up .image quadlets
Yeah you just specify the name of the .image file in the .container file
And quadlet wires it up as a dependency
Then the pull becomes the service and it doesn't time our starting the container service

Set up template files
you should prefer the template module when you're trying to control file content entirely, rather than blockinfile where you want to edit a block within a larger file that you're not trying to own with ansible

Daemon reload handler

Set up array for pod

How to run podman has another user:  
`sudo -u {{ podman_user }} bash -li`

Dedicate a private git repo just for that content and version it and make it easy for the dev to spin up a test instance for himself and use tagged references to deploy to prod from git

We get two volumes, one for html and one for the SQL database. 
```bash
$ podman volume ls
DRIVER      VOLUME NAME
local       testsite
local       testsitedb

```

You can see the contents of the volume with podman volume inspect under mount point:
```bash
seal@wordpress-01:~/.local/share/containers/storage/volumes/testsite/_data$ podman volume inspect testsite
[
     {
          "Name": "testsite",
          "Driver": "local",
          "Mountpoint": "/home/seal/.local/share/containers/storage/volumes/testsite/_data",
          "CreatedAt": "2025-12-17T10:43:45.347003912-07:00",
          "Labels": {},
          "Scope": "local",
          "Options": {},
          "MountCount": 0,
          "LockNumber": 4
     }
]
```

Now that the container has generated the volume, we can copy the html files from `/home/seal/.local/share/containers/storage/volumes/testsite/_data`over to our `ansible/config/wordpress-01/` directory:
```bash
rsync -av . dthomas@sombrero:~/Ansible/config/wordpress-01/html/
```

We'll need to do the same for the database volume:
```bash
podman volume inspect testsitedb
[
     {
          "Name": "testsitedb",
          "Driver": "local",
          "Mountpoint": "/home/seal/.local/share/containers/storage/volumes/testsitedb/_data",
          "CreatedAt": "2025-12-17T10:43:45.35804307-07:00",
          "Labels": {},
          "Scope": "local",
          "Options": {},
          "MountCount": 0,
          "NeedsCopyUp": true,
          "NeedsChown": true,
          "LockNumber": 5
     }
]
```

Rsync the files over to the control node:


1. And make it easy for the dev to spin up a copy of the container, that isn't prod, using uncomitted changes
    
2. So he can iterate before you push to prod
    
3. he could even do it on his workstation with Podman Desktop and you could roll him a little recipe to do that
    
4. so he doesn't need to SSH to some server to do it

### Customize config
#### Example from httpd (convert this to work with wordpress

To customize the configuration of the httpd server, first obtain the upstream default configuration from the container:

```console
$ docker run --rm httpd:2.4 cat /usr/local/apache2/conf/httpd.conf > my-httpd.conf
```

You can then `COPY` your custom configuration in as `/usr/local/apache2/conf/httpd.conf`:

```dockerfile
FROM httpd:2.4
COPY ./my-httpd.conf /usr/local/apache2/conf/httpd.conf
```
