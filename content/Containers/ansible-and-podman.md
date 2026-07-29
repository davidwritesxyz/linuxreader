---
title: Ansible and Podman
description: Ansible and podman quadlets
---

If you're used to managing packages installed on VMs, it's definitely a step up to run containers in podman via quadlet units

ansible + podman is pretty close to k8s in terms of overall "how easy is this to manage"-ness, just not in terms of your RTO times or your overall scalability or resiliency

You would totally use ansible to generate the necessary configuration file(s) via jinja templates, confirm the desired state of your runtime/selinux/firewalls/OS/whatever,  and then `containers.podman.podman_container` or `containers.podman.podman_play`  or whatever