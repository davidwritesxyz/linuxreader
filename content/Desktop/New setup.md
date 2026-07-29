---
description: "New Fedora Build Using Ansible" 
title: "Configure Fedora Desktop using Ansible" 
---

```bash
sudo dnf -y install vim

### Make vim default sudoer editor
echo "Defaults editor=/usr/bin/vim" | sudo tee /etc/sudoers.d/99_custom_editor

### remove password prompts when using sudo
sudo sed -i 's/^#\s*%wheel\s\+ALL=(ALL)\s\+NOPASSWD: ALL/%wheel  ALL=(ALL) NOPASSWD: ALL/' /etc/sudoers
sudo sed -i 's/^%wheel\s\+ALL=(ALL)\s\+ALL/# %wheel  ALL=(ALL) ALL/' /etc/sudoers

sudo dnf -y install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
    https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

sudo dnf5 install 'dnf5-command(groupinstall)'

sudo dnf -y groupinstall \
      "Development Tools"

/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.bash_profile
    eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"

sudo dnf -y install ansible
```

ansible setup
`vim setup.yml`

```yaml
---
- name: Setup Development Environment
  hosts: localhost
  become: yes
  tasks:

    # Install Flatpak applications
    - name: Install Flatpak applications
      flatpak:
        name: "{{ item }}"
        state: present
      loop:
        - com.bitwarden.desktop
        - com.brave.Browser
        - org.gimp.GIMP
        - org.gnome.Snapshot
        - org.libreoffice.LibreOffice
        - org.remmina.Remmina
        - com.termius.Termius
        - com.slack.Slack
        - org.keepassxc.KeePassXC
        - md.obsidian.Obsidian
        - com.calibre_ebook.calibre
        - org.mozilla.Thunderbird
        - us.zoom.Zoom
        - org.wireshark.Wireshark
        - com.google.Chrome
        - io.github.shiftey.Desktop
        - io.github.dvlv.boxbuddyrs
        - com.github.tchx84.Flatseal
        - io.github.flattool.Warehouse
        - io.missioncenter.MissionCenter
        - com.github.rafostar.Clapper
        - com.mattjakeman.ExtensionManager
        - com.jgraph.drawio.desktop
        - org.adishatz.Screenshot
        - com.github.finefindus.eyedropper
        - com.github.johnfactotum.Foliate
        - com.obsproject.Studio
        - com.vivaldi.Vivaldi
        - com.vscodium.codium
        - io.podman_desktop.PodmanDesktop
        - org.kde.kdenlive
        - org.virt_manager.virt-manager
        - io.github.input_leap.input-leap
        - com.nextcloud.desktopclient.nextcloud

    # Install Development Tools group using dnf
    - name: Install Development Tools group
      dnf:
        name: "@Development Tools"
        state: present

    - name: Install @virtualization group package
      dnf:
        name: '@virtualization'
        state: present

    # Update dnf configuration
    - name: Update dnf configuration for fastestmirror and parallel downloads
      block:
        - lineinfile:
            path: /etc/dnf/dnf.conf
            line: "fastestmirror=True"
        - lineinfile:
            path: /etc/dnf/dnf.conf
            line: "max_parallel_downloads=10"
        - lineinfile:
            path: /etc/dnf/dnf.conf
            line: "defaultyes=True"
        - lineinfile:
            path: /etc/dnf/dnf.conf
            line: "keepcache=True"

    # Perform DNF update and install required packages
    - name: Update DNF and install required packages
      dnf:
        name:
          - gnome-screenshot
          - wireguard-tools
          - gnome-tweaks
          - gnome-themes-extra
          - telnet
          - nmap
        state: present

    # Set GNOME theme (using gsettings directly)
    - name: Set GNOME theme to Adwaita-dark
      shell: gsettings set org.gnome.desktop.interface gtk-theme "Adwaita-dark"
      become_user: "davidt"

    - name: Enable experimental Mutter features
      shell: gsettings set org.gnome.mutter experimental-features "['scale-monitor-framebuffer']"
      become_user: "davidt"

    # Install Go programming language
    - name: Install Go
      dnf:
        name: go
        state: present

    - name: Add Go to the PATH in .bashrc
      lineinfile:
        path: "/home/davidt/.bashrc"
        line: 'export PATH=$PATH:/usr/local/go/bin'
        state: present
      become_user: "davidt"

    - name: Source .bashrc
      shell: source /home/davidt/.bashrc
      become_user: "davidt"
      
    - name: Install pip using yum
      yum:
        name: python-pip
        state: present
    
```

run the playbook:
`ansible-playbook setup.yml`

Then reboot...

Then sign into nextcloud and begin sync.

 # Install Homebrew packages:
 `brew install hugo`

Install gnome extentions:
```bash
pip install --user gnome-extensions-cli
gext install "appindicatorsupport@rgcjonas.gmail.com"
gext enable "appindicatorsupport@rgcjonas.gmail.com"
gext install "legacyschemeautoswitcher@joshimukul29.gmail.com"
gext install "blur-my-shell@aunetx"
gext install "dash-to-dock@micxgx.gmail.com"
gext install "gsconnect@andyholmes.github.io"
gext install "logomenu@aryan_k"
gext install "search-light@icedman.github.com"
```

Restore remmina connections
`cp ~/Nextcloud/remmina/* ~/.var/app/org.remmina.Remmina/data/remmina/`

Restore vimrc
`cat ~/Nextcloud/Documents/dotfiles/vimrc.bak > ~/.vimrc`

Restore ~/.bashrc: (if username is the same)
`cat ~/Nextcloud/Documents/dotfiles/bashrc.bak > ~/.bashrc`

Git config
```bash
git config --global user.email "tdavetech@gmail.com"
git config --global user.name "linuxreader"

# Store git credentials (from inside a git directory):
git config credential.helper store
```

## OneDrive

Install:
`sudo dnf -y install onedrive`

Start:
`onedrive`

Display config:
```
onedrive --display-config
```

Sync and prefer local copy:
`onedrive --sync --local-first`

Enable the user level service:
`onedrive --user enable --now onedrive

Force local to the cloud
onedrive --synchronize --force

Restore files from cloud
onedrive --synchronize --resync

Add foce option to top of the user service file to ignore big delete flag. 
`systemctl --user edit onedrive`

```
[Service]
ExecStart=
ExecStart=/usr/bin/onedrive --monitor --verbose --force
```