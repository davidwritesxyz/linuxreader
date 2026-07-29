---
title: 'Host Name Patterns'
description: 'Working with host name patterns'
showDate: false
---

## Working with Host Name Patterns

- If you want to use an IP address in a playbook, the IP address must be specified as such in the inventory. 
- You cannot use IP addresses that are based only on DNS name resolving. 
- So specifying an IP address in the playbook but not in the inventory file---assuming DNS name resolution is going to take care of the IP address resolving---doesn't work.

- Apart from the specified groups, there are the implicit host groups all and ungrouped. 
- Host name wildcards may be used. 
	- `ansible -m ping 'ansible\*'`
		- Match all hosts that have a name starting with ansible. 
		- Must put the pattern between single quotes or it will fail with a no matching hosts error. 
	- Can be used at any place in the host name. 
		- `ansible -m ping '\*ble1'`

- When you use wildcards to match host names, Ansible doesn't distinguish between IP addresses, host names, or hosts; it just matches anything. 
	- `'web\*'` 
		- Matches all servers that are members of the group 'webservers', but also hosts 'web1' and 'web2'.

To address multiple hosts:
- You specify a comma-separated list of targets to address multiple hosts:
	- `ansible -m ping ansible1,192.168.4.202`
	- Can be a mix of host names, IP addresses, and host group names.

Operators:
- Can specify a logical AND condition by including an ampersand (&), and a logical NOT by using an exclamation point (!). 
	- `web,&file` applies to hosts only if they are members of the web and file groups
	- `web,!webserver1` applies to all hosts in the web group, except host webserver1. 
	- When you use the logical AND operator, the position of the ampersand doesn't matter.
		- `web,&file` as `&web,file` also. 
	- You can use a colon (:) instead of a comma (,), but using a comma is better to avoid confusion when using IPv6 addresses.

