We are sick of manually setting up virtual machines and configuring them. You can use Ansible to automate the configuration of a server. But what about setting up a server before it's configured? Let's stop doing that manually as well by using Vagrant.

Also, by setting up your infrastructure as code, you get to set up machines quickly and consistently manage and deploy your servers. This reduces the time it takes, operating costs, and minimizes the chance of configuration errors. 

I just spent a full hour setting up and Ansible lab by manually attaching an installation disk, configuring IPs, selectng software, setting up accounts, etc. 

![](/images/Pasted%20image%2020250401042551.png)

Vagrant uses a configuration file called a *vagrantfile*. This file has all of the information needed to deploy a new virtual machine. Let's install Vagrant on Fedora:
```bash
sudo dnf install -y dnf-plugins-core
sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/fedora/hashicorp.repo
sudo dnf -y install vagrant
```

Install remaining dependencies and plugin:
```bash
sudo dnf install --assumeyes libvirt libguestfs-tools \
  gcc libvirt-devel libxml2-devel make ruby-devel
vagrant plugin install vagrant-libvirt
```

You will also need virt-manager, libvirt, and qemu for this tutorial. 
![](../../images/Pasted%20image%2020250401043118.png)


First, we need a plugin that will let Vagrant interact with our VM provider. Install libvirt plugin to do this:
`vagrant plugin install vagrant-libvirt`

Libvirt documenttation can be found [here](https://vagrant-libvirt.github.io/vagrant-libvirt/).

And another plugin to manage VM disk sizes:
`vagrant plugin install vagrant-disksize`


`cd ~/Documents/projects`

### Setting up a Git repository for version control and sharing.
In Github, make a new repository, I will call mine "vagrant-vms". Then clone the repo to your local host:
```bash
cd ~/Documents/projects
git clone https://github.com/linuxreader/vagrant-vms.git
cd vagrant-vms
```

### Vagrantfile

A vagrantfile descibes how to build and provision a VM. A vagrant file is written in Ruby. To make a vagrant file, you need to add a box using the vagrant command, then initialize the box. Vagrant boxes are OS images that Vagrant provides [here.](https://portal.cloud.hashicorp.com/vagrant/discover)

```bash
david@fedora:~/Documents/projects/vagrant-vms$mkdir ansible-lab
david@fedora:~/Documents/projects/vagrant-vms$ cd ansible-lab
david@fedora:~/Documents/projects/vagrant-vms/ansible-lab$ vagrant box add almalinux/9
==> box: Loading metadata for box 'almalinux/9'
    box: URL: https://vagrantcloud.com/api/v2/vagrant/almalinux/9
This box can work with multiple providers! The providers that it
can work with are listed below. Please review the list and choose
the provider you will be working with.

1) hyperv
2) libvirt
3) parallels
4) virtualbox
5) vmware_desktop

Enter your choice: 2
==> box: Adding box 'almalinux/9' (v9.5.20241203) for provider: libvirt
    box: Downloading: https://vagrantcloud.com/almalinux/boxes/9/versions/9.5.20241203/providers/libvirt/amd64/vagrant.box
    box: Calculating and comparing box checksum...
==> box: Successfully added box 'almalinux/9' (v9.5.20241203) for 'libvirt'!
```

Then, initialize the box:
```bash
$ vagrant init almalinux/9

A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```

This will place a vagrant file into the current directory:
```bash
$ ls
Vagrantfile

$ cat Vagrantfile 
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "almalinux/9"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end
```

For this project, we are going to set up an Ansible lab. With 1 control VM, and 2 host VMs. 

#### Control
name: control
Ip: 192.168.124.200
RAM: 2048
Storage: 20GB

I could not find a way to increase the initial disk capacity but there is a guide on changing the disk size after it's been made: https://nullr0ute.com/2018/08/increasing-a-libvirt-kvm-virtual-machine-disk-capacity/

Add an additional disk
```ruby
libvirt.storage :file, :size => '20G'
```

You can use either the command line option `--provider=kvm` or you can set the `VAGRANT_DEFAULT_PROVIDER` environment variable:

```bash
export VAGRANT_DEFAULT_PROVIDER=libvirt  # <-- may be in ~/.profile, /etc/profile, or elsewhere

vagrant up
```