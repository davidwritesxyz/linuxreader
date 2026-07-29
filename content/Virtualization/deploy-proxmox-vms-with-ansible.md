---
draft: true
date: 2026-05-04
title: How to Automate Proxmox VM Creation with Ansible
tags: ["articles"]
summary: "How to Automate Proxmox VM Creation with Ansible"
---
## Set up API

![](../../../../images/Pasted%20image%2020260504053309.png)

Note the token id and secret
davidt@pam!ansible
4bf81b0d-330f-433d-a4ea-6a9ce37a9ee0

## Set up proxmox server for ansible
```
root@proxmox:~# adduser davidt
New password: 
Retype new password: 
passwd: password updated successfully
Changing the user information for davidt
Enter the new value, or press ENTER for the default
	Full Name []: 
	Room Number []: 
	Work Phone []: 
	Home Phone []: 
	Other []: 
Is the information correct? [Y/n] 
```

```
root@proxmox:~# apt install sudo
```

`visudo`

```
davidt ALL=(ALL:ALL) NOPASSWD: ALL
```

