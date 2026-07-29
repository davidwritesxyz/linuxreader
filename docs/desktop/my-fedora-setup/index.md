# My old Fedora Setup

When you first download Fedora Workstation, it's going to be a little hard to figure out how to make it usable. Especially if you've never tinkered with Linux before.

This is because Fedora with Gnome desktop is a blank canvas. The point is to let you customize it to your needs. When I first install Fedora, I pull my justfile to install most of the programs I use:

	`curl -sL https://raw.githubusercontent.com/linuxreader/dotfiles/main/dot_justfile -o ~/.justfile`

## Install just and run the justfile

To run the just file, I then install the just program and run it on the justfile:

`dnf install just`

`just first-install`

This is my current .justfile:
```bash
first-install:
# Install flatpacks
	flatpak install --noninteractive \
      flathub com.bitwarden.desktop \
      flathub com.brave.Browser \
      flathub org.gimp.GIMP \
      flathub org.gnome.Snapshot \
      flathub org.libreoffice.LibreOffice \
      flathub org.remmina.Remmina \
      flathub com.termius.Termius \
      flathub com.slack.Slack \
      flathub org.keepassxc.KeePassXC \
      flathub md.obsidian.Obsidian \
      flathub com.calibre_ebook.calibre \
      flathub org.mozilla.Thunderbird \
      flathub us.zoom.Zoom \
      flathub org.wireshark.Wireshark \
      flathub com.nextcloud.desktopclient.nextcloud \
      flathub com.google.Chrome \
      flathub io.github.shiftey.Desktop \
      flathub io.github.dvlv.boxbuddyrs \
      flathub com.github.tchx84.Flatseal \
      flathub io.github.flattool.Warehouse \
      flathub io.missioncenter.MissionCenter \
      flathub org.gnome.World.PikaBackup \
      flathub com.github.rafostar.Clapper \
      flathub com.mattjakeman.ExtensionManager \
      flathub com.jgraph.drawio.desktop \
      flathub org.adishatz.Screenshot \
      flatpak com.github.finefindus.eyedropper \
      flatpak com.github.johnfactotum.Foliate \
      flatpak com.usebottles.bottles \
      flatpak com.obsproject.Studio \
      flatpak net.lutris.Lutris \
      flatpak com.vivaldi.Vivaldi \
      flatpak com.vscodium.codium \
      flatpak io.podman_desktop.PodmanDesktop \
      flatpak org.kde.kdenlive

# Install Homebrew
    sudo dnf -y groupinstall \
      "Development Tools"

    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.bash_profile
    eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"

# Configure dnf for faster speeds
    sudo bash -c 'echo "fastestmirror=True" >> /etc/dnf/dnf.conf'
    sudo bash -c 'echo "max_parallel_downloads=10" >> /etc/dnf/dnf.conf'
    sudo bash -c 'echo "defaultyes=True" >> /etc/dnf/dnf.conf'
    sudo bash -c 'echo "keepcache=True" >> /etc/dnf/dnf.conf'

# Other software, updates, etc. 
    sudo dnf -y update
    sudo dnf install -y gnome-screenshot
    sudo dnf -y groupupdate core
    sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
    sudo dnf install -y wireguard-tools
    sudo dnf install gnome-tweaks
    sudo dnf -y install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
    https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
    sudo dnf -y update
    sudo dnf install gnome-themes-extra
    gsettings set org.gnome.desktop.interface gtk-theme "Adwaita-dark"
    sudo dnf install -y go
    echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
    source ~/.bashrc
    gsettings set org.gnome.mutter experimental-features "['scale-monitor-framebuffer']"

homebrew:
    brew install \
      chezmoi \
      hugo \
      virt-manager
```

## VSCodium
Added for starship compatibility
```
❯ cat ~/.config/starship.toml 
[container]
disabled = true
```

`❯ cp /usr/bin/starship ~/.local/bin/starship`

Add to end of bashrc
```
#eval "$(starship init bash)"
if [ -z "$CONTAINER_ID" ]; then
    eval "$(starship init bash)"
fi
```

Settings.json
```
{
    "window.titleBarStyle": "custom",
    "editor.fontFamily": "'Cascadia Code', 'Droid Sans Mono', 'monospace', monospace, 'Symbols Nerd Font Mono'",
    "update.mode": "none",
    "workbench.editor.empty.hint": "hidden",
    "explorer.confirmDragAndDrop": false,
    "editor.autoClosingBrackets": "never",
    "editor.suggestOnTriggerCharacters": false,
    "editor.quickSuggestions": {
        "comments": "off",
        "strings": "off",
        "other": "off"
    },
    "editor.hover.enabled": false,
    "editor.parameterHints.enabled": false,
    "editor.suggest.snippetsPreventQuickSuggestions": false,
    "html.suggest.html5": false,
    "terminal.integrated.profiles.linux": {
        "bash": {
            "path": "/bin/bash",
            "args": ["--login", "-i"]
        }
    },
    "terminal.integrated.defaultProfile.linux": "bash"
}
```
## Install Homebrew Stuff:

then run `just homebrew` after a reboot to install packages with brew


### visudo config
Add to /etc/sudoers to make Vim default for `visudo`

`Defaults editor=/usr/bin/vim` 



## Virt Manager
```bash
sudo dnf install @virtualization
```

```bash
sudo vi /etc/libvirt/libvirtd.conf
```

Uncomment the line: `unix_sock_group = "libvirt"`

Adjust the UNIX socket permissions for the R/W socket:
`unix_sock_rw_perms = "0770"`

Start the service:
`systemctl enable --now libvirtd`

Add user to group:
```bash
sudo usermod -a -G libvirt $(whoami) && sudo usermod -a -G kvm $(whoami)
```

use the Tweaks app to set the appearance of Legacy Applications to 'adwaita-dark'.
## Configure Howdy 
Howdy is a tool for using an IR webcam for authentication:
```bash
sudo dnf copr enable principis/howdy
sudo dnf --refresh install -y howdy
```

https://copr.fedorainfracloud.org/coprs/principis/howdy/
https://github.com/boltgolt/howdy

## Seahorse
I was using this to fix the Login Keyring error that is common with Fedora, but it no longer works.
`sudo dnf -y install seahorse && seahorse`

Applications > Passwords and Keys > Passwords > Right-click Login > Change Password to blank.

https://itsfoss.com/seahorse/

## Initialize Chezmoi
Chezmoi let's you easy sync your dotfiles with Github and your other computers. Just init Chezmoi and add your Github username. This assumes your dotfiles in Github are saved in the proper format. 
`chezmoi init --apply linuxreader

## Add badname user (if needed)
If you need to use username with the format `firstname.lastname`, use the `badname` flag with the `adduser` command. You will have to create a normal user first, because you can't do this during the initial install:
`$ adduser --badname firstname.lastname`
`$ sudo usermod -aG wheel username`

```
# uncomment this line in the visudo file 
$ sudo visudo
%wheel ALL=(ALL) ALL
```

Delete the other user:
`$ userdel username`

## Additional DNF stuff:
Clear cache (do this occasionally):
`sudo dnf clean dbcache`
or `sudo dnf clean all`

Update DNF:
`sudo dnf -y update`

Additional DNF commands:
[https://docs.fedoraproject.org/en-US/fedora/latest/system-administrators-guide/package-management/DNF/](https://docs.fedoraproject.org/en-US/fedora/latest/system-administrators-guide/package-management/DNF/ "https://docs.fedoraproject.org/en-US/fedora/latest/system-administrators-guide/package-management/DNF/")

## Set up RPM Fusion
RPM Fusion give you more accessibility to various software packages.

Install:
```
sudo dnf -y install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

[https://rpmfusion.org/](https://rpmfusion.org/ "https://rpmfusion.org/")
## AppStream metadata
Use AppStream to enable users to install packages using Gnome Software/KDE Discover:
```
sudo dnf -y groupupdate core
```

## Flatpaks
To enable Flatpaks (this may no longer be needed:
```
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

https://flatpak.org/setup/Fedora
## Set a hostname
Set a hostname for the system. This will show after next reboot:
`sudo hostnamectl set-hostname "New_Custom_Name"`


## Other Stuff
Here is some other stuff I install from the software center.

### Input Leap
Input Leap let's you share a mouse and keyboard between two workstations.

![](/images/Pasted%20image%2020240905105932.png)

I don't know what this is or why it is here:
installing Git and a bunch of other stuff?
```bash
sudo dnf install git cmake make gcc-c++ xorg-x11-server-devel \
                 libcurl-devel avahi-compat-libdns_sd-devel \
                 libXtst-devel qt5-qtbase qt5-qtbase-devel  \
                 qt5-qttools-devel libICE-devel libSM-devel \
                 openssl-devel libXrandr-devel libXinerama-devel
```

## Virtual machine Manager
The best way to manage VMs on desktop.

![](/images/Pasted%20image%2020240905110024.png)

## Box Buddy
For managing containers.

![](/images/Pasted%20image%2020240905110223.png)
## Warehouse
Managing installed applications.

![](../../images/Pasted%20image%2020240905110143.png)

## Mission Center 
Task Manager like application.

![](/images/Pasted%20image%2020240905110500.png)

## Pika Backup
For backing up your desktop.
![](/images/Pasted%20image%2020240905110555.png)
## Clapper
Video player.

![](/images/Pasted%20image%2020240905110319.png)

And some extensions installed through Extension Manager:
![](/images/Pasted%20image%2020240905105517.png)

Install Gnome Screenshot tool:

Install the extention:
https://extensions.gnome.org/extension/1112/screenshot-tool/

You also need to install from DNF for some reason:
`dnf install -y gnome-screenshot`

## Airpods not pairing Issue
If you ever have the issue where Airpods won't pair. Remove them from the pairing list, force them in pairing mode, and pair them back. This can be made easy with bluetoothctl:

Just in case, restart the bluetooth service:
```bash
sudo systemctl restart bluetooth
systemctl status bluetooth 
```

Show devices:
```
# bluetoothctl

[bluetooth] $ devices
Device 42:42:42:42:42:42 My AirPods <-- grab the name here
[bluetooth] $ remove 42:42:42:42:42:42 
```

Now, make sure your Airpods are in the charging case, close the lid, wait 15 seconds, then open the lid. Press and hold the setup button on the case for up to 10 seconds. The status light should flash white, which means that your Airpods are ready to connect.

Pair them back:
```
[bluetooth] $ pair 42:42:42:42:42:42
```

## Enable Fractional Scaling
This lets you change display scaling in smaller increments. You'll need to make sure Wayland is turned on.

Turn on the feature then reboot:
`gsettings set org.gnome.mutter experimental-features "['scale-monitor-framebuffer']"`

`reboot`

## Remove pasted characters ^[](200~')
Kept having pasted format characters ^[](200~') mess up my groove. Here's the fix.

Open your inputrc file and add the line below:
`vim ~/.inputrc`

```bash
"\C-v": ""
```

## Install gimp
`sudo dnf install gimp`
enable in screenshot tool after install for gimp {f}

Install Arrows
https://graphicdesign.stackexchange.com/questions/44797/how-do-i-insert-arrows-into-a-picture-in-gimp

```
Go to your home folder
Go to .config/GIMP
Go to the folder with a version number (2.10 for me)
Go to scripts
Download the arrow.scm file and place it here. Don't forget to unzip.
Open GIMP and draw a path
```

From Tools menu, select Arrow

= h.265 main 10 profile media codec error =


## Distrobox

See [distrobox](../Containers/distrobox.md) 

## ZSH For Humans

[GitHub - romkatv/zsh4humans: A turnkey configuration for Zsh](https://github.com/romkatv/zsh4humans)

## Install Starcraft on Fedora

https://www.youtube.com/watch?v=eefsL9K2w4k

 1. Install your latest gpu driver https://github.com/lutris/docs/blob/master/InstallingDrivers.md

I am just running off of built in AMD graphics. So we just need to install support for Vulkan API
`sudo dnf install vulkan-loader vulkan-loader.i686`


Install Wine
`$ sudo dnf -y install wine`

Install Lutris
Install the Flatpak version in software center.

## Fedora Hotkeys

## Terminal

Close Terminal
shift + c + q

Previous Tab
c + Page Up

Next Tab
c + Page Down

Move to Specific Tab
Alt + \#

Full Screen
F11

New Window
Shift + Ctrl + t

Close Tab
Shift + Ctrl + w


## Desktop

Run a command
super + F2

Switch Between Applications
Alt + Esc

Move Window to Left Monitor
Shift + Super + <- 

Move Window to Right Monitor
Shift + Super + ->

Minimize Current Window
Super + H

Close Current Appllication
Ctrl + Q

## Browser

### Firefox

Switch Between Tabs
Ctrl + Tab

Switch Between Tabs in Reverse
Ctrl + Shift + Tab

#### Detach Tab Extension
https://addons.mozilla.org/en-US/firefox/addon/detach-tab/

Detach Tab
Ctrl _ Shift _ Space

Reattach Tab
Ctrl + Shift + v

## Slack

Installing via package manager because of screen sharing issue. 

Upgrade dnf and download the slack rpm from the website. 

**Screen Sharing in Slack:**

```bash
vim /usr/share/applications/slack.desktop
```

Update the exec line to:
```bash
Exec=/usr/bin/slack --enable-features=WebRTCPipeWireCapturer %U
```

## Actual Budget

https://github.com/actualbudget/actual

### Set Vim to default editor for visudo
add `Defaults editor=/usr/bin/vim` to top of visudo file.su