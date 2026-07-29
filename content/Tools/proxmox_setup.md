Add the following to the end of /etc/apt/sources.list
```
# not for production use
deb http://download.proxmox.com/debian/pve bullseye pve-no-subscription
```
You may need to replace this for your current version. See https://pve.proxmox.com/wiki/Package_Repositories

This will download "unstable" updates for proxmox. 

Next, open pve-enterprise.list if you do not have a subscription. This lets you avoid subscription errors.

```
vim /etc/apt/sources.list.d/pve-enterprise.list
```

Then add a "#" to comment out this line:
\# deb https://enterprise.proxmox.com/debian/pve bullseye pve-enterprise
```
Then, update the lists:
```
apt-get update
```

Next, Run a distribution upgrade and reboot:
```
apt dist-upgrade
reboot
``

## Make the network bridge vlan aware
network > select bridge > edit > check "vlan aware"