---
title: 'Execution Environments'
description: 'Ansible Execution Environments'
draft: true
---
Why use EEs?
- Portable Ansible environments
	- includes Ansible core version
	- All desired collections
	- Python dependencies
	- Bind dependencies
	- Anything you need to run a playbook

A container that has a specific version of Ansible. Can test execution in a specific Ansible environment to make sure it will work with that version. 

EEs are built leveraging ansible-bulder
They can be pushed to a private automation hub or any container registry
Run EEs from the cli using ansible-navigator
Or run in your production environment using automation controller as part of the Ansible Automation Platform
If you want them to automatically occur, schedule them as a job inside AAP