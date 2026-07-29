---
title: 'Using Vagrant on Linux'
description: 'How to use Vagrant from a Linux computer.'
---

Vagrant is software that lets you set up multiple, pre-configured virtual machines in a flash. I am going to show you how to do this using Linux and Virtual Box. But you can do this on MacOS and Windows as well.

Download Vagrant, VirtualBox and Git.

[Vagrant](https://www.vagrantup.com/downloads) link.

[Virtualbox](https://www.virtualbox.org/wiki/Downloads) link.

You may want to follow another [tutorial](https://www.youtube.com/watch?v=8mns5yqMfZk) for setting up VirtualBox.

[Git](https://git-scm.com/download/win) link.

Installing git will install ssh on windows. Which you will use to access your lab. Just make sure you select the option to add git and unit tools to your PATH variable.

Make a Vagrant project folder.

Note: All of these commands are going to be in a Bash command prompt.
```bash
mkdir vagranttest
```

Move in to your new directory.
```bash
cd vagranttest
```

Add and Initialize Your Vagrant Project.

You can find preconfigured virtual machines [here](https://app.vagrantup.com/boxes/search).

We are going to use ubuntu/trusty64.


Add the Vagrant box
```bash
vagrant box add ubuntu/trusty64
```

Initialize your new Vagrant box
```bash
vagrant init ubuntu/trusty64
```


Use the dir command to see the contents of this directory.


We are going to edit this Vagrantfile to set up our multiple configurations.

```bash
vim Vagrantfile
```

Here is the new config without all of the commented lines. Add this (minus the top line) under Vagrant.configure("2") do |config|.

```bash
Vagrant.configure("2") do |config|  
  config.vm.box = "ubuntu/trusty64"  
  config.vm.define "server1" do |server1|  
    server1.vm.hostname = "server1"  
    server1.vm.network "private_network", ip: "10.1.1.2"  
  end  
  config.vm.define "server2" do |server2|  
    server2.vm.hostname = "server2"  
    server2.vm.network "private_network", ip: "10.1.1.3"  
  end  
end
```

Now save your Vagrant file in Vim.

Bring up your selected vagrant boxes:

```bash
vagrant up
```

Now if you open virtual box, you should see the new machines running in headless mode. This means that the machines have no user interface..


Ssh into server1

```bash
vagrant ssh server1
```

You are now in serve1's terminal.

From server1, ssh into server2

```bash
ssh 10.1.1.3
```

Success! You are now in server2 and can access both machines from your network. Just enter "exit" to return to the previous terminal.

Additional Helpful Vagrant Commands.

Without the machine name specified, vagrant commands will work on all virtual machines in your vagrant folder. I've thrown in a couple examples using [machine-name] at the end.

Shut down Vagrant machines

```bash
vagrant halt
```

Shut down only one machine

```bash
vagrant halt [machine-name]
```

Suspend and resume a machine

```bash
vagrant suspend
```

```bash
vagrant resume
```

Restart a virtual machine

```bash
vagrant reload
```

Destroy a virtual machine

```bash
vagrant detstroy [machine-name]
```

Show running vms

```bash
vagrant status
```

List Vagrant options

```bash
vagrant
```

Playground for future labs

This type of deployment is going to be the bedrock of many Linux and Red Hat labs. You can easily use pre-configured machines to create a multi-machine environment. This is also a quick way to test your network and server changes without damaging anything.

Now go set up a Vagrant lab yourself and let me know what you plan to do with it!

## What is Vagrant?

- Easy to configure, reproducible environments
- Provisions virtualbox vms
- Vagrant box: OS image

Syntax:
```bash
vagrant box add user/box
```

Add centos7 box
```bash
vagrant box add jasonc/centos7
```
Many public boxes to download

Vagrant project = folder with a vagrant file

Install Vagrant here: [https://www.vagrantup.com/downloads](https://www.vagrantup.com/downloads "https://www.vagrantup.com/downloads")

Make a vagrant folder:
```bash
mkdir vm1
cd vm1
```    

initialize vagrant project:
```bash
vagrant init jasonc/centos7
```    

bring up all vms defined in the vagrant file)
```bash
vagrant up
```    

vagrant will import the box into virtualbox and start it

the vm is started in headless mode

(there is no user interfaces)

Vagrant up / multi machine

Bring up only one specific vm

-   vagrant up [vm-name]

SSH Vagrant

-   vagrant ssh [vm_name] or vagrant ssh if there is only one vm in the vagrant file

Need to download ssh for windows

downloading git will install this:

[https://desktop.github.com/](https://desktop.github.com/ "https://desktop.github.com/")

Shut down vagrant machines
`vagrant halt`

Shutdown only one machine
`vagrant halt [vm]`

Saves present state of the machine

just run vagrant up without having to import tha machines again

Suspend the machine
`vagrant suspend [VM]`

Resume
`vagrant resume [VM]`


Destroy VM
`vagrant destroy [VM]`

List options
`vagrant`

Vagrant command works on the vagrant folder that you are in

Vagrant File

```
Vagrant.configure (2) do | config |

config.vm.box = "jasonc/centos7"

config.vm.hostname = "linuxsvr1"

(default files)

config.vm.network "private_network", ip: "10.2.3.4"

config.vm.provider "virtualbox" do | vbi

vb.gui = true

vb.memory = "1024"

(shell provisioner)

config.vm.provision "shell", path: "setup.sh"

end

end
```

Configuring a multi machine setup:

Specify common configurations at the top of the file

```
Vagrant.configure (2) do | config |

config.vm.box = "jasonc/centos7"

config.vm.define = "server1" do | server1 |

server1.vm.hostname = "server1"

server1.vm.network "private_network", ip: "10.2.3.4"

end

config.vm.define = "server2" do | server2 |

server2.vm.hostname = "server2"

server2.vm.network "private_network", ip: "10.2.3.5"

end

end
```

You can search for vagrant boxes at [https://app.vagrantup.com/boxes/search](https://app.vagrantup.com/boxes/search "https://app.vagrantup.com/boxes/search")

Course software downloads: [http://mirror.linuxtrainingacademy.com/](http://mirror.linuxtrainingacademy.com/ "http://mirror.linuxtrainingacademy.com/")

Install Git: [https://git-scm.com/download/win](https://git-scm.com/download/win "https://git-scm.com/download/win")

- make sure to check option for git and unit tools to be added to the PATH

`vagrant ssh`
    
-   to connect to the vagrant machine in the folder that you are in    
-   default password in vagrant  
-   tyoe 'exit to return to  prompt

`vagrant halt`
-   stop the vm and save it's current state

`vagrant reload`
-   restarts the vm

`vagrant status`
-   shows running vms in that folder

You can access files in the vagrant directory from both VMs

## Example RHEL8 Config


```bash
Vagrant.configure("2") do |config|

config.vm.box = "generic/rhel8"

config.vm.define "server1" do |server1|

server1.vm.hostname = "server1.example.com"

server1.vm.network "private_network", ip: "192.168.1.110"

config.disksize.size = '10GB'

end

config.vm.define "server2" do |server2|

server2.vm.hostname = "server2.example.com"

server2.vm.network "private_network", ip: "192.168.1.120"

config.disksize.size = '16GB'

end

config.vm.provider "virtualbox" do |vb|

vb.memory = "2048"

end

end
```

Plugin to change the disk size:
```bash
vagrant plugin install vagrant-disksize
```


