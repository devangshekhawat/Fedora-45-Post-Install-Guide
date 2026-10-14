 
# Fedora 45 Post Install Guide
Things to do after installing Fedora 45

## RPM Fusion

* Fedora has disabled the repositories for a lot of free and non-free .rpm packages by default. Follow this if you want to use non-free software like Steam, Discord and some multimedia codecs etc. As a general rule of thumb it is advised to do this to get access to many mainstream useful programs.
* Enable third party repositories by pasting the following into the terminal: 
* `sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm`
* also while you're at it, install app-stream metadata by:
* `sudo dnf install -y rpmfusion-free-appstream-data rpmfusion-nonfree-appstream-data`

  

## Update 
* Go into the software center and click on update. Alternatively, you can do:
* `sudo dnf -y update`
* Reboot

## Firmware
* If your system supports firmware update delivery through lvfs, update your device firmware by:
```
fwupdmgr refresh --force
fwupdmgr get-devices # Lists devices with available updates.
fwupdmgr get-updates # Fetches list of available updates.
fwupdmgr update
```

## Flatpak
* Fedora doesn't include all non-free flatpaks by default. In-case you forgot to check the "Enable Third Party Repositories" option on initial boot, the command below enables access to all the flathub flatpaks. 
* `flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo`
* Disable Fedora's Flatpak repository, as we already have the Flathub's official one enabled. Having more than one Flatpak repository makes installing software unnecessary complex.
* `flatpak remote-modify --disable fedora`

## AppImage

* For Appimage support install fuse:
* `sudo dnf install fuse-libs`
* You can also install an AppImage manager like [Gearlever](https://flathub.org/apps/it.mijorus.gearlever) for neater management. To do so, run the following command:
* `flatpak install it.mijorus.gearlever` 

## NVIDIA Drivers
* `sudo dnf update` #To make sure you're on the latest kernel and then reboot.
* Check if you have Secure Boot enabled with: `mokutil --sb-state`. If enabled, follow number 1 if not then number 2

<details>
<summary>(A) Secure Boot Enabled:</summary>
Follow these steps if you have Secure Boot Enabled:

1.  Install the tools below to work with the keys:
 * `sudo dnf install kmodtool akmods mokutil openssl`
2.  Generate a key with default values:
 * `sudo kmodgenca -a`
 *  Note: If you see "WARNING: EXISTING KEY PAIR", then add `--force` at the end of the command and run it again.
3.  Enroll the key with the command below:
 * `sudo mokutil --import /etc/pki/akmods/certs/public_key.der`
 * When prompted create a short simple password like "1234" and remember it for a later step.
 * Reboot by typing `systemctl reboot`
4. In the blue screen that shows up on startup, do:
 * `"Enroll MOK" -> "Continue" -> "Yes" -> "Enter Password (i.e. 1234)"`
 * Reboot Again
5. Once logged in, open the terminal and install Nvidia drivers:
 * `sudo dnf install akmod-nvidia`
 * For CUDA support install the package below (if you use Davinci Resolve, Blender etc.)
 * `sudo dnf install xorg-x11-drv-nvidia-cuda`
 * Wait for atleast 5 mins before rebooting, to let the kernel module get built. Once its built, the command above would output the driver version instead of an error.
 * `modinfo -F version nvidia # check if kmod is built` 
 * Reboot once its built.
* Congrats now you have working Nvidia drivers with secure boot enabled!
</details>

<details>
<summary>(B) Secure Boot Disabled </summary>
Install Nvidia drivers by following the steps below in a terminal:


 * `sudo dnf install akmod-nvidia`
 * For CUDA support install the package below (if you use Davinci Resolve, Blender etc.)
 * `sudo dnf install xorg-x11-drv-nvidia-cuda`
 * Wait for atleast 5 mins before rebooting, to let the kernel module get built. Once its built, the command above would output the driver version instead of an error.
 * `modinfo -F version nvidia # check if kmod is built` 
 * Reboot once its built.
 * Congrats now you have working Nvidia drivers!
</details>

* Note (optional): If your disk is encrypted follow the Encrypted Disk section of [this guide](https://github.com/Comprehensive-Wall28/Nvidia-Fedora-Guide?tab=readme-ov-file#encrypted-drives) .

## ~~Battery Life (Deprecated)~~
* ~~Follow this if you have a Laptop and are facing sub optimal battery backup.~~
* ~~power-profiles-daemon which come pre-configured on fedora works well on a great majority of systems but still in case you're facing sub-optimal battery backup you try installing tlp by:~~
* ~~`sudo dnf install tlp tlp-rdw`~~
* ~~and mask power-profiles-daemon by:~~
* ~~`sudo systemctl mask power-profiles-daemon`~~
* ~~Also install powertop by:~~
* ~~`sudo dnf install powertop`~~
* ~~`sudo powertop --auto-tune`~~
* Edit: Fedora comes preinstalled with [Tuned](https://fedoraproject.org/wiki/Changes/TunedAsTheDefaultPowerProfileManagementDaemon) which works well on its own now and all the aforementioned changes are now unnecessary. Just follow [HW video acceleration](https://github.com/devangshekhawat/Fedora-40-Post-Install-Guide/blob/main/README.md#hw-video-acceleration) for better battery backup. 

## Media Codecs
* Install these to get proper multimedia playback.
````
sudo dnf swap 'ffmpeg-free' 'ffmpeg' --allowerasing # Switch to full FFMPEG.
sudo dnf update @multimedia --setopt="install_weak_deps=False" --exclude=PackageKit-gstreamer-plugin --exclude=libheif-freeworld --exclude=obs-studio-freeworld # Update multimedia/GStreamer components while excluding currently broken broken packages.
sudo dnf group install -y sound-and-video # Installs useful Sound and Video complementary packages.
````

## H/W Video Acceleration
* Helps decrease load on the CPU when watching videos online by alloting the rendering to the dGPU/iGPU. Quite helpful in increasing battery backup on laptops.

### H/W Video Decoding with VA-API 
* `sudo dnf install ffmpeg-libs libva libva-utils`

<details>
<summary>Intel</summary>
 
* If you have a recent Intel chipset (5th Gen and above) after installing the packages above., Do:
* `sudo dnf swap libva-intel-media-driver intel-media-driver --allowerasing`
* `sudo dnf install libva-intel-driver`
</details>

<details>
<summary>AMD</summary>
 
No need to do this for intel integrated graphics. Mesa drivers are for AMD graphics, who lost support for h264/h245 in the fedora repositories in f38 due to legal concerns.
 
* If you have an AMD chipset, after installing the packages above do:
```
sudo dnf install mesa-va-drivers-freeworld
sudo dnf install mesa-va-drivers-freeworld.i686
```
</details>

### OpenH264 for Firefox
* `sudo dnf install -y openh264 gstreamer1-plugin-openh264 mozilla-openh264`
* `sudo dnf config-manager setopt fedora-cisco-openh264.enabled=1`
* After this enable the OpenH264 Plugin in Firefox's settings.

## Set Hostname
* `hostnamectl set-hostname YOUR_HOSTNAME`

## Default Firefox start page 
* The tweak below will make the start page the default firefox start page instead of [this](https://fedoraproject.org/start), (Note: If you're using Fedora Silverblue and not stock Fedora then click [here](https://github.com/devangshekhawat/Fedora-44-Post-Install-Guide/issues/49#issuecomment-4172103035))
* `sudo rm -f /usr/lib64/firefox/browser/defaults/preferences/firefox-redhat-default-prefs.js`

## Custom DNS Servers
* For people that want to setup custom DNS servers for better privacy
```
sudo mkdir -p '/etc/systemd/resolved.conf.d' && sudo -e '/etc/systemd/resolved.conf.d/99-dns-over-tls.conf'

[Resolve]
DNS=1.1.1.2#security.cloudflare-dns.com 1.0.0.2#security.cloudflare-dns.com 2606:4700:4700::1112#security.cloudflare-dns.com 2606:4700:4700::1002#security.cloudflare-dns.com
DNSOverTLS=yes
Domains=~.
```

## Set UTC Time
* Used to counter time inconsistencies in dual boot systems
* `sudo timedatectl set-local-rtc '0'`

## Optimizations
* The tips below can allow you to squeeze out a little bit more performance from your system. 

### Disable `NetworkManager-wait-online.service`
* Disabling it can decrease the boot time by at least ~15s-20s:
* `sudo systemctl disable NetworkManager-wait-online.service`

### Disable Gnome Software from Startup Apps
* Gnome software autostarts on boot even though it is not required on every boot, this takes at least 100MB of RAM upto 900MB (as reported anecdotically). Unless you want it to do search for updates in the background or give you application suggestions in GNOME overview,  you can stop it from autostarting by:
* `mkdir -p ~/.config/autostart`
* `cp /usr/share/applications/org.gnome.Software.desktop ~/.config/autostart/`
* `echo "X-GNOME-Autostart-enabled=false" >> ~/.config/autostart/org.gnome.Software.desktop`
* GNOME Software is also triggered in the background whenever you search in the GNOME overview. To prevent this, disable it as a search provider:
* `dconf write /org/gnome/desktop/search-providers/disabled "['org.gnome.Software.desktop']"`

## Gnome Extensions [Optional]
* Suggestions for good utilities to extend the capabilities of your system
* Don't install these if you are using a different spin of Fedora.
* Pop Shell - run `sudo dnf install -y gnome-shell-extension-pop-shell xprop` to install it.
* [GSconnect](https://extensions.gnome.org/extension/1319/gsconnect/) - run `sudo dnf install nautilus-python` for full support. then `sudo firewall-cmd --permanent --zone=public --add-service=kdeconnect`
* [Gesture Improvements](https://extensions.gnome.org/extension/4345/gesture-improvements/)
* [Quick Settings Tweaker](https://github.com/qwreey75/quick-settings-tweaks)
* [User Themes](https://extensions.gnome.org/extension/19/user-themes/)
* [Compiz Windows Effect](https://extensions.gnome.org/extension/3210/compiz-windows-effect/)
* [Just Perfection](https://extensions.gnome.org/extension/3843/just-perfection/)
* [Rounded Windows Corners](https://extensions.gnome.org/extension/5237/rounded-window-corners/)
* [Dash to Dock](https://extensions.gnome.org/extension/307/dash-to-dock/)
* [Quick Settings Tweaker](https://extensions.gnome.org/extension/5446/quick-settings-tweaker/)
* [Blur My Shell](https://extensions.gnome.org/extension/3193/blur-my-shell/)
* [Bluetooth Quick Connect](https://extensions.gnome.org/extension/1401/bluetooth-quick-connect/)
* [App Indicator Support](https://extensions.gnome.org/extension/615/appindicator-support/)
* [Clipboard Indicator](https://extensions.gnome.org/extension/779/clipboard-indicator/)
* [Legacy (GTK3) Theme Scheme Auto Switcher](https://extensions.gnome.org/extension/4998/legacy-gtk3-theme-scheme-auto-switcher/)
* [Caffeine](https://extensions.gnome.org/extension/517/caffeine/)
* [Vitals](https://extensions.gnome.org/extension/1460/vitals/)
* [Wireless HID](https://extensions.gnome.org/extension/4228/wireless-hid/)
* [Logo Menu](https://extensions.gnome.org/extension/4451/logo-menu/)
* [Space Bar](https://github.com/christopher-l/space-bar)

## Apps [Optional]
* Packages for Rar and 7z compressed files support:
 `sudo dnf install -y unzip p7zip p7zip-plugins unrar`
* These are Some Packages that I use and would recommend:
```
7zip
Amberol
Blanket
Builder
Brave 
Blender
Discord
Drawing
Deja Dup Backups
Endeavour 
Easyeffects
Extension Manager
Flatseal
Foliate
Footage
GIMP
Gnome Tweaks
Gradience
Handbrake
Iotas
Joplin
Khronos
Krita
Logseq
lm_sensors
Onlyoffice
Overskride
Parabolic
Pcloud
PDF Arranger
Planify
Pika backup 
Snapshot
Solanum
Sound Recorder
Tangram
Transmission
Ulauncher
Upscaler
Video Trimmer
VS Codium
yt-dlp
```
  
## Theming [Optional]

### GTK Themes
* Don't install these if you are using a different spin of Fedora.
* https://github.com/lassekongo83/adw-gtk3
* https://github.com/vinceliuice/Colloid-gtk-theme
* https://github.com/EliverLara/Nordic
* https://github.com/vinceliuice/Orchis-theme
* https://github.com/vinceliuice/Graphite-gtk-theme

### Use themes in Flatpaks
* `sudo flatpak override --filesystem=$HOME/.themes`
* `sudo flatpak override --env=GTK_THEME=my-theme` 

### Icon Packs
* https://github.com/vinceliuice/Tela-icon-theme
* https://github.com/vinceliuice/Colloid-gtk-theme/tree/main/icon-theme

### Wallpaper
* https://github.com/manishprivet/dynamic-gnome-wallpapers

### Firefox Theme
* Install Firefox Gnome theme by: `curl -s -o- https://raw.githubusercontent.com/rafaelmardojai/firefox-gnome-theme/master/scripts/install-by-curl.sh | bash`

### Starship (terminal theme)
* Configure starship to make your terminal look good (refer https://starship.rs)

### Grub Theme
* https://github.com/vinceliuice/grub2-themes
