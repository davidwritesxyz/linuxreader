1. libvirt is the most mature and proven virtualization stack in the world
    
2. proxmox is just a management layer on top of it
    
3. so is kubevirt
    
4. so are like a dozen other things
qemu is a general purpose linux virtualization swiss army knife CLI, the way most people use it most of the time is as an interface to KVM which is just built into the kernel

5. KVM is what is actually running most of your VMs, qemu is a cli for interacting with it, libvirt is a family of APIs and services for managing those interactions in a more declarative way, PVE is a... family of APIs and services for managing those interactions in a more declarative way (lol)
6. kubevirt is a way to bridge from the k8s API down to libvirt interactions on nodes on your cluster
qemu does other stuff, too - i use it sometimes to do architecture emulation, which isn't using KVM

Commands to manage Libvirt:

`virsh`
- Manage VMs