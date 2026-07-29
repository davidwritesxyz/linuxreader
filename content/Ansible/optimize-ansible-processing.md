---
title: 'Optimizing Ansible Processing'
description: 'Optimizing Ansible Processing'
showDate: false
---

## Optimizing Ansible Processing 


**Parallel task execution** 
- Manages the number of hosts on which tasks are executed simultaneously. 
**Serial task execution** 
- Tasks are executed on a host or group of hosts before proceeding to the next host or group of hosts.

#### Parallel Task Execution
 
- Ansible can run tasks on all hosts at the same time, and in many cases that would not be a problem because processing is executed on the managed host anyway. 
- If, however, network devices or other nodes that do not have their own Python stack are involved, processing needs to be done on the control host. 
- To prevent the control host from being overloaded in that case, the maximum number of simultaneous connections by default is set to 5. 
- You can manage this setting by using the **forks** parameter in ansible.cfg. 
- Alternatively, you can use the `-f` option with the `ansible` and `ansible-playbook` commands.
- If only Linux hosts are managed, there is no reason to keep the maximum number of simultaneous tasks much lower than 100.

#### Managing Serial Task Execution
- While executing tasks, Ansible processes tasks in a playbook one by one.
- This means that, by default, the first task is executed on all managed hosts. Once that is done, the next task is processed, until all tasks have been executed. 
- There is no specific order in the execution of tasks, so you may see that in one run ansible1 is processed before ansible2, while on another run they might be processed in the opposite order.
- In some cases, this is undesired behavior. 
- If, for instance, a playbook is used to update a cluster of hosts this way, this would create a situation where the old software has been updated, but the new version has not been started yet and the entire cluster would be down. 
- Use the **serial** keyword in the play header to configure
	- **serial: 3** 
		- All tasks are executed on three hosts, and after completely running all tasks on three hosts, the next group of three hosts is handled. 

### Lab: Managing Parallelism

- Add two more managed nodes with the names ansible3.example.com and ansible4.example.com. 
- Open the inventory file with an editor and add the following lines:

``` pre1
ansible3
ansible4
```

- Open the ansible.cfg file and add the line `forks = 4` to the **[defaults]** section.
- Write a playbook with the name **exercise102-install** that installs and enables the Apache web server and another playbook with the name **exercise102-remove** that disables and removes the Apache web server.
- Run `ansible-playbook exercise102-remove.yaml` to remove and disable the Apache web server on all hosts. This is just to make sure you start with a clean configuration.
- Run the playbook to install and run the web server, using `time ansible-playbook exercise102-install.yaml`, and notice the time it takes to run the playbook.
- Run `ansible-playbook exercise102-remove.yaml` again to get back to a clean state.
- Edit **ansible.cfg** and change the **forks** parameter to `forks = 2`. 
- Run the `time ansible-playbook exercise102-install.yaml` command again to see how much time it takes now
- Edit the `exercise102-install.yaml` playbook and include the line `serial: 2` in the play header.
- Run the `ansible-playbook exercise102-remove.yaml` command again to get back to a clean state.
- Run the `ansible-playbook exercise102-install.yaml` command again and observe that the entire play is executed on two hosts only before the next group of two hosts is taken care of.

