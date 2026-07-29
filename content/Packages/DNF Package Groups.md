---
title: DNF Package Groups
summary: All about DNF Package groups
---

A package group  is a group of packages that serve a common purpose. This let's you query, install, and delete multiple packages as a single unit.

There are two types of package groups: _environment groups_ and _package groups_. 

**environment groups** are the option listed on the software selection window during RHEL installation. They provide all the necessary software to build the OS for a desired purpose. Environment groups include: 
- Server
- Server with GUI
- Minimal Install
- Workstation
- KDE Plasma Workspaces
- Virtualization Host
- Custom Operating System

**Package groups** are groups of packages that serve a common purpose.They help save time on the deployment of individual and dependent packages. These include:
- Container Management
- Headless Management
- Console Internet Tools
- .NET Development
- RPM Development Tools
- Development Tools
- Graphical Administration Tools
- Legacy UNIX Compatibility
- Network Servers
- Scientific Support
- Security Tools
- Smart Card Support
- System Tools
- Fedora Packager
- VideoLAN Client
- Xfce
## Package Group Management

Use the `group` dnf subcommand to list, install, query, and remove groups of packages.

### Listing Available and Installed Package Groups

List all available and installed package groups from all repositories:
```bash
 dnf group list
```

The output shows both environment and package groups.
```bash
Available Environment Groups:
   Server with GUI
   Minimal Install
   Workstation
   KDE Plasma Workspaces
   Virtualization Host
   Custom Operating System
Installed Environment Groups:
   Server
Installed Groups:
   Container Management
   Headless Management
Available Groups:
   Console Internet Tools
   .NET Development
   RPM Development Tools
   Development Tools
   Graphical Administration Tools
   Legacy UNIX Compatibility
   Network Servers
   Scientific Support
   Security Tools
   Smart Card Support
   System Tools
   Fedora Packager
   VideoLAN Client
   Xfce
```

Show packages included in a given package group:
```bash
sudo dnf group info "Scientific Support"
Last metadata expiration check: 2:04:48 ago on Tue 02 Jun 2026 07:21:12 AM MST.
Group: Scientific Support
 Description: Tools for mathematical and scientific computations, and parallel computing.
 Optional Packages:
   atlas
   fftw
   fftw-devel
   fftw-static
   lapack
   mpich-devel
   openmpi
   openmpi-devel
   python3-numpy
   python3-scipy
   units
```

Show all available package groups:
```bash
dnf group list
```

List installed package groups:
```bash
sudo dnf group list --installed
Last metadata expiration check: 2:07:40 ago on Tue 02 Jun 2026 07:21:12 AM MST.
Installed Environment Groups:
   Server
Installed Groups:
   Container Management
   Headless Management
```

### Installing and Updating Package Groups

When you install a package group dnf will create the directory structure for all the packages included in the group. Along with any dependent packages. It will install required files and run any required post installation tasks. It will also update all packages in the group to the latest version if they aren't already. 

Install the "Scientific Support" package group:
```bash
sudo dnf group install "Scientific Support"
Last metadata expiration check: 0:02:51 ago on Tue 02 Jun 2026 09:39:23 AM MST.
Dependencies resolved.
===============================================================================================
 Package               Architecture         Version                Repository             Size
===============================================================================================
Installing Groups:
 Scientific Support                                                                           

Transaction Summary
===============================================================================================

Is this ok [y/N]: y
Complete!
```

To update it to the latest version:
```bash
sudo dnf group update "Scientific Support"
Last metadata expiration check: 0:07:38 ago on Tue 02 Jun 2026 09:39:23 AM MST.
Dependencies resolved.
===============================================================================================
 Package               Architecture         Version                Repository             Size
===============================================================================================
Upgrading Groups:
 Scientific Support                                                                           

Transaction Summary
===============================================================================================

Is this ok [y/N]: y
Complete!
```

### Removing Package Groups
Removing a package group uninstalls all packages and dependencies included in the group. It also deletes associated files and directory structure.

Erase all evidence of the "Scientific Support" package group:
```bash
sudo dnf group remove "Scientific Support"
Dependencies resolved.
===============================================================================================
 Package               Architecture         Version                Repository             Size
===============================================================================================
Removing Groups:
 Scientific Support                                                                           

Transaction Summary
===============================================================================================

Is this ok [y/N]: y
Complete!

```
