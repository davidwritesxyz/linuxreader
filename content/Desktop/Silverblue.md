---
title: "Silverblue"
description: "Fedora Silverblue" 
---

Automated setup https://universal-blue.org/

You get all the benefits of using containers
Separates system level packages from applications.

System Level
- Desktop, kernel?
- Layering
	- Apps at system level because containers aren't as developed yet
	- Locks to the fedora version you are on

## Layered package examples
	- gnome shell extensions
	- distrobox

Uses rpm-ostree? https://coreos.github.io/rpm-ostree/administrator-handbook/

Flatpacks

Remove fedora flatpack stuff and use flathub repos instead https://flatpak.org/setup/Fedora

Systemd unit for automatic flatpack updates

Update every 4 hours to mirror ubuntu

flatseal
adjust permissions of flatpacks

check out apps.gnome.org

## Rebase into Universal Blue

Rebase onto the "unsigned" image then reboot:
`rpm-ostree rebase ostree-unverified-registry:ghcr.io/ublue-os/silverblue-main:39` and 

Then the signed image and reboot:
`rpm-ostree rebase ostree-image-signed:docker://ghcr.io/ublue-os/silverblue-main:39`
    
Then do we you do after install, open the app store and install stuff via GUI or we 

## just

have a .justfile to install all flatpak/ homebrew packages
https://universal-blue.discourse.group/t/introduction-to-just/42
https://just.systems/man/en/chapter_1.html

My justfile
```bash
import "/usr/share/ublue-os/justfile"  
# You can add your own commands here! For documentation, see: [https://ublue.it/guide/just/](https://ublue.it/guide/just/)  
  
first-install:  
    flatpak install \  
      flathub com.bitwarden.desktop \  
      flathub com.brave.Browser \  
      flathub com.discordapp.Discord \  
      flathub net.cozic.joplin_desktop \  
      flathub org.gimp.GIMP \  
      flathub org.gnome.Snapshot \  
      flathub org.libreoffice.LibreOffice \  
      flathub org.remmina.Remmina \  
      flathub com.termius.Termius \  
      flathub net.devolutions.RDM \  
      flathub com.slack.Slack \  
      flathub org.keepassxc.KeePassXC \  
      flathub md.obsidian.Obsidian \  
      flathub com.calibre_ebook.calibre \  
      flathub com.logseq.Logseq \  
      flathub org.mozilla.Thunderbird \  
      flathub us.zoom.Zoom \  
      flathub org.wireshark.Wireshark \  
      flathub com.nextcloud.desktopclient.nextcloud \  
      flathub com.google.Chrome  
  
    brew install \  
      ansible \  
      chezmoi \  
      neovim  \      
      onedrive \  
      wireguard-tools
```

Set up github dotfiles repo

Install chezmoi and initialize:
`chezmoi init`

Sync with Chezmoi: https://www.chezmoi.io/quick-start/

Add dotfiles
`chezmoi add ~/.bashrc`

Edit a dotfile
`chezmoi edit ~/.bashrc`

See changes
`chezmoi diff`

Apply changes
`chezmoi -v apply`

to sync chezmoi with git:
```bash
chezmoi cd
git remote add origin https://github.com/$GITHUB_USERNAME/dotfiles.git 
$ git push -u origin main 
$ exit
```

For subsequent git pushes:
```bash
git commit -a -m "commit" && git push
```

![](/images/Pasted%20image%2020240904141011.png)

### From a second machine:

Install all dotfiles with a single command:
`chezmoi init --apply https://github.com/$GITHUB_USERNAME/dotfiles.git`

If you use GitHub and your dotfiles repo is called `dotfiles` then this can be shortened to:
`$ chezmoi init --apply $GITHUB_USERNAME`


See a list of full commands:
`chezmoi help`

Or you can initialize and choose what you want:
`chezmoi init https://github.com/$GITHUB_USERNAME/dotfiles.git`

See what changes are awaiting:
`chezmoi diff`

Apply changes:
`chezmoi apply -v`

can also edit a file before applying:
`chezmoi edit $FILE`

Or merge the current file with new file:
`chezmoi merge $FILE`

From any machine, you can pull and apply changes from your repo:
`chezmoi update -v`

![](../../images/Pasted%20image%2020240904141728.png)

Add the justfile:
`chezmoi add .justfile`

## Install Connect Tunnel

Download from website
Install java
`rpm-ostree install java`

Runn the connect tunnel install script

Commands located in /var/usrlocal/Aventail must be ran as root
`sudo ./startctui.sh

