---
title: 'Ansible Builder'
description: 'Using Ansible Builder to build an Execution Environment'
draft: true
---

Build portable control nodes packaged as containers. (Execution environments)
- Works with AWX and Ansible Navigator for playbook development and testing.
- Able to choose specific Python and Ansible-core version
- Also package with Python packages, system packages, and Ansible collections. 

Steps needed:
1. Install `ansible-builder`
2. Make sure podman is installed
3. Make an execution-environment.yml file that includes:
	1. Base container image
	2. Python version
	3. Ansible-core version
	4. ansible-runner version
	5. collections with version restrictions
	6. system packages with version restrictions
	7. Python packages with version restrictions
	8. other items to download, intsall, or configure

- If base image includes Python you omit that. 

Ansible builder execute's two steps:
1. create containerfile for podman or Dockerfile for docker based on the definition file
2. run containerization tool to build an image based on the build instruction file and build context

`ansible-builder build`
- runs both steps

`ansible-builder create`
- runs first step only

#### Building images with `ansible-builder`

Four stages to build a container image:
1. Base: pull the base image, installPython version, pip, ansible-runner, and ansible-core
2. Galaxy: download collections and store them locally as files
3. Builder: download python/system packages and store them locally as files
4. Final: install downloaded files on the output of the base stage and generating a new image that includes all the content. 

Ansible Builder injects hooks at each stage of the container build process so you can add custom steps before and after every build stage.

You may need to install certain packages or utilities before the Galaxy and Builder stages. For example, if you need to install a collection from GitHub, you must install git after the Base stage to make it available during the Galaxy stage.

To add custom build steps, add an `additional_build_steps` section to your execution environment definition.

Install:
` pip3 install ansible-builder`

