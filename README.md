<div align="center">

[![Author](https://img.shields.io/badge/XETRA-585b70)](https://github.com/wtfxetra)
[![License](https://img.shields.io/badge/License-Apache--2.0-blue)](./LICENSE.txt)
![Last Updated](https://img.shields.io/badge/Last_Updated-January_2026-585b70)

</div>

<h1 align="center">Void Linux Installation Guide x86_64 [AMD]</h1>

<div align="center">
    <i>How to install Void Linux</i>
</div>

### Getting Started

Welcome to the Void Linux Installation Guide!

This guide provides you with a step-by-step walkthrough of installing
Void Linux with the Wayland and Niri Compositor. It has been created
based on my own installation of Void Linux.

### Support and Feedback

If you have any suggestions, corrections, or encounter any issues while following
the guide, I encourage you to get involved through Github.


<h1 align="center">
    Section 01: Step-by-step guide of installing Void Linux on your hardware
</h1>

### Step 01: Downloading Void Linux image

1. Go to Void Linux downloads page https://voidlinux.org/download/

2. Find Base GlibC live image matching your system architecture .

### Step 02: Preparing installation medium

1. Insert a USB-stick into your PC with at least 1.5Gb of space available on it.

2. Find block device for USB-stick in `/dev` folder. Usually it is `/dev/sdb`.

<dl><dd>
<b>IMPORTANT NOTE</b>: you need block device without a number on the end.
If you have for example <i>/dev/sdb</i>, <i>/dev/sdb1</i> and <i>/dev/sdb2</i> you need <i>/dev/sdb</i> !
</dd></dl>

3. Burn previously downloaded Void Linux ISO-image on a USB-stick (in my case it is `/dev/sdb`):

<dl><dd>
<pre>
$ <b>sudo dd status=progress bs=4M if=./voidlinux-YYYYMMDD-x86_64-base.iso of=/dev/sdb</b>
</pre>
</dd></dl>

### Step 03: Boot into Void Linux medium

1. Insert the installation medium into the computer on which you are installing Void Linux.

2. Power on your PC and press _boot menu key_. For _HP laptops,
   this key is `F9`. For _Framework its `F12`.

3. Boot from USB-stick and wait until boot process is finished.

<dl><dd>
<b>IMPORTANT NOTE</b>: not every device can run a system from USB-stick out of the box.
Many BIOS'es by default come with activated <i>Secure boot</i> option.You might need to
deactivate it in your BIOS.
</dd></dl>

### Step 04: Connect to Network
1. Ethernet Users Skip This.
1. Connect to WiFi using `wpa_supplicant` and check connection is established:

<dl><dd>
<pre>
$ <b>wpa_cli</b>
 > <b>scan</b>
<> <b>scan_results</b>
<> <b>add_network</b>
<> <b>set_network 0 ssid "\"YourWiFiName\""</b>
<> <b>set_network 0 psk "\"YourWiFiPassword\""</b>
<> <b>enable_network 0</b>
<> <b>exit</b>
$ <b>ping voidlinux.org</b>
</pre>
</dd></dl>

### Step 05: Disk partitioning
1. Wipe Drive. You can find storage device name using `lsblk` command.

<dl><dd>
<pre>
$ <b>wipefs -a /dev/nvme0n1</b>
</pre>
</dd></dl>

2. Partition main storage device using `fdisk` utility.

<dl><dd>
<pre>
$<b>fdisk /dev/nvme0n1</b>
                <i>[create GPT Layout]</i>
Command (m for help): <b>g</b>
Created a new GPT disklabel (GUID: A7A02FDD-365E-4498-A226-91327326FDF0).
                <i>[create partition 1: efi]</i>
Command (m for help): <b>n</b>
Partition number (1-128, default 1): <b>Enter &crarr;</b>
First sector (..., default 2048): <b>Enter &crarr;</b>
Last sector ...: <b>+512M</b>
<span />
                <i>[create partition 2: root]</i>
Command (m for help): <b>n</b>
Partition number (3-128, default 2): <b>Enter &crarr;</b>
First sector (..., default ...): <b>Enter &crarr;</b>
Last sector ...: <b>Enter &crarr;</b>
<span />
                <i>[change partitioning type]</i>
Command (m for help): <b>t</b>
Partition number (1-2, default 1): <b>1</b>
Partion type or alias (type L to list all): <b>uefi</b>
Command (m for help): <b>t</b>
Partition number (1-2, default 2): <b>2</b>
Partion type or alias (type L to list all): <b>linux</b>
<span />
                <i>[write partitioning to disk]</i>
Command (m for help): <b>w</b>
</pre>
</dd></dl>

2. Create filesystems on created disk partitions:

<dl><dd>
<pre>
$ <b>mkfs.fat -F 32 -n voidboot /dev/nvme0n1p1</b> <i># on EFI System partition</i>
$ <b>mkfs.btrfs -L voidroot /dev/nvme0n1p2</b>   <i># on Linux filesystem partition</i>
</pre>
</dd></dl>

3. Correctly mount all filesystems to the `/mnt`:

<dl><dd>
<pre>
$ <b>mount /dev/nvme0n1p2 /mnt</b>
$ <b>btrfs subvolume create /mnt/@</b>
$ <b>btrfs subvolume create /mnt/@home</b>
$ <b>btrfs subvolume create /mnt/@var</b>
$ <b>btrfs subvolume create /mnt/@snapshots</b>
$ <b>umount /mnt/</b>
$ <b>mount -o defaults,noatime,space_cache=v2,compress-force=zstd:3,commit=90,subvol=@ /dev/nvme0n1p2 /mnt </b>
$ <b>mkdir -p /mnt/{home,boot/efi,var,snapshots}</b>
$ <b>mount -o subvol=@home /dev/nvme0n1p2 /mnt/home</b>
$ <b>mount -o subvol=@var /dev/nvme0n1p2 /mnt/var</b>
$ <b>mount -o subvol=@snapshots /dev/nvme0n1p2 /mnt/snapshots</b>
$ <b>chattr +C /mnt/var</b>
$ <b>mount /dev/nvme0n1p1 /mnt/boot/efi</b>
</pre>
</dd></dl>

4. Install essential packages into new filesystem and generate fstab:

<dl><dd>
<pre>
$ <b>XBPS_ARCH=x86_64 xbps-install -Sy -r /mnt -R "https://repo-default.voidlinux.org/current" base-system vim iwd seatd</b>
$ <b>xgenfstab -U /mnt > /mnt/etc/fstab</b>
</pre>
</dd></dl>

### Step 06: Basic configuration of new system

1. Chroot into freshly created filesystem:

<dl><dd>
<pre>
$ <b>xchroot /mnt /bin/bash</b>
</pre>
</dd></dl>

2. Setup system locale and timezone, sync hardware clock with system clock:

<dl><dd>
<pre>
$ <b>vim /etc/default/libc-locales</b>   <i># uncomment your locales, i.e. `en_US.UTF-8`</i>
$ <b>locale-gen</b>
$ <b>ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime</b>   <i># choose your timezone</i>
$ <b>hwclock --systohc</b>
$ <b>vim /etc/rc.conf
  #Uncomment and Configure these
  KEYMAP="us"
  FONT="lat9w-16"
  #Save and Exit
</pre>
</dd></dl>

3. Setup system hostname:

<dl><dd>
<pre>
$ <b>echo <i>yourhostname</i> > /etc/hostname</b>
</pre>
</dd></dl>

4. Add new users and setup passwords:

<dl><dd>
<pre>
$ <b>useradd -m -G wheel,storage,audio,video,optical,users,_seatd yourusername</i></b>
$ <b>passwd root</b>
$ <b>passwd <i>yourusername</i></b>
</pre>
</dd></dl>

5. Add wheel group to sudoers file to allow users to run sudo:

<dl><dd>
<pre>
$ <b>EDITOR=vim visudo</b>
    <i>[uncomment following line in file]</i>
    <i>%wheel ALL=(ALL) ALL</i>
</pre>
</dd></dl>

6. Install GRUB:

<dl><dd>
<pre>
$ <b>xbps-install -Sy grub-x86_64-efi</b>
$ <b>grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Grub</b>
</pre>
</dd></dl>

7. Configure System:

<dl><dd>
<pre>
$ xbps-reconfigure -fa
</pre>
</dd></dl>

9. Exit chroot, unmount all disks and reboot:

<dl><dd>
<pre>
$ <b>exit</b>
$ <b>umount -R /mnt</b>
$ <b>reboot</b>
</pre>
</dd></dl>

<h1 align="center">
    Section 02: Configuring userspace after initial system setup
</h1>

### Step 01: Basic configuration of userspace

1. Activate services:

<dl><dd>
<pre>
$ <b>sudo ln -sv /etc/sv/acpid /var/service</b>
$ <b>sudo ln -sv /etc/sv/dhcpcd /var/service</b>
$ <b>sudo ln -sv /etc/sv/dbus /var/service</b>
$ <b>sudo ln -sv /etc/sv/iwd /var/service</b>
$ <b>sudo ln -sv /etc/sv/seatd /var/service</b>

$ logout
</pre>
### Login Back
</dd></dl>

2. [Connect to WiFi using `iwd`:

<dl><dd>
<pre>
$ iwctl
[iwd]# station list 
                            Devices in Station Mode                            
--------------------------------------------------------------------------------
  Name                  State            Scanning
--------------------------------------------------------------------------------
  wlo1                  connected                   
[iwd]# station wlo1 scan
[iwd]# station wlo1 get-networks 
                               Available networks                              
--------------------------------------------------------------------------------
      Network name                      Security            Signal
--------------------------------------------------------------------------------
      WIFI_NAME 2                        psk                 ****      
[iwd]# station wlo1 connect "WIFI_NAME 2"
[iwd]# exit
$
</pre>
</dd></dl>

3. Install Wayland and its utilities:

<dl><dd>
<pre>
$ <b>sudo xbps-install wayland mesa mesa-dri vulkan-loader mesa-vulkan-loader</b>
</pre>
</dd></dl>

4. Install a bunch of useful utilities:

<dl><dd>
<pre>
$ <b>sudo xbps-install void-repo-nonfree</b>              <i># Void NonFree Repo</i>
$ <b>sudo xbps-install lshw</b>              <i># Provides detailed information on the hardware of the machine</i>
$ <b>sudo xbps-install inxi</b>              <i># Full featured CLI system information tool</i>
$ <b>sudo xbps-install acpi</b>              <i># Client for battery, power, and thermal readings</i>
<div><div/>
$ <b>sudo xbps-install base-devel</b>        <i># Basic tools to build Void Linux packages</i>
$ <b>sudo xbps-install git</b>               <i># Distributed version control system</i>
$ <b>sudo xbps-install zip</b>               <i># Compressor/archiver for creating and modifying zipfiles</i>
$ <b>sudo xbps-install unzip</b>             <i># For extracting and viewing files in .zip archives</i>
$ <b>sudo xbps-install 7zip</b>             <i># For extracting and viewing files in .7z archives</i>
$ <b>sudo xbps-install btop</b>              <i># Interactive CLI process viewer</i>
$ <b>sudo xbps-install lsd</b>              <i># A directory listing program</i>
<div><div/>
$ <b>sudo xbps-install dialog</b>            <i># A tool to display dialog boxes from shell scripts</i>
$ <b>sudo xbps-install bash-completion</b>   <i># Programmable completion for the bash shell</i>
<div><div/>
$ <b>sudo xbps-install net-tools</b>         <i># Configuration tools for Linux networking</i>
$ <b>sudo xbps-install ethtool</b>           <i># Utility for controlling network drivers and hardware</i>
$ <b>sudo xbps-install wget</b>              <i># Network utility to retrieve files from the Web</i>
$ <b>sudo xbps-install curl</b>              <i># Network utility to retrieve files from the Web</i>
$ <b>sudo xbps-install rsync</b>             <i># File copying tool for remote and local files</i>
</pre>
</dd></dl>

5. Install Niri and Dependencies:

<dl><dd>
<pre>
<i># Instructions for installing Niri</i>
<div></div>
## Wayland / Niri Setup (Void Linux)

$ sudo xbps-install niri
$ sudo xbps-install nautilus
$ sudo xbps-install waybar          # nice statusbar for wayland
$ sudo xbps-install fuzzel          # like dmenu, but more customizable
$ sudo xbps-install alacritty       # terminal emulator
$ sudo xbps-install foot            # lightweight terminal emulator
$ sudo xbps-install fnott           # notification manager
$ sudo xbps-install swww            # fast and light wallpaper utility
$ sudo xbps-install xdg-desktop-portal-gtk   # desktop portal
$ sudo xbps-install xdg-user-dirs   # user dir setup
$ sudo xbps-install yazi            # console file manager

---

## Graphics Stack (Common)


$ sudo xbps-install mesa
$ sudo xbps-install mesa-dri
$ sudo xbps-install vulkan-loader

---

## For AMD GPUs


$ sudo xbps-install mesa-vulkan-radeon
$ sudo xbps-install mesa-vaapi
$ sudo xbps-install mesa-vdpau


---

## For Intel GPUs


$ sudo xbps-install mesa-vulkan-intel
$ sudo xbps-install intel-video-accel
---

<div></div>
<div></div>
</pre>
</dd></dl>


6. Install essential system fonts:

<dl><dd>
<pre>
$ <b>sudo xbps-install ttf-dejavu ttf-freefont ttf-liberation ttf-droid terminus-font</b>
$ <b>sudo xbps-install noto-fonts noto-fonts-emoji ttf-ubuntu-font-family ttf-roboto ttf-roboto-mono ttf-ibm-plex</b>
</pre>
</dd></dl>

7. Enable sound support on your PC:

<dl><dd>
<pre>
$ <b>sudo xbps-install sof-firmware</b>    # Sound Open Firmware
$ <b>sudo xbps-install alsa-utils</b>      # Advanced Linux Sound Architecture - Utilities
$ <b>sudo xbps-install alsa-plugins</b>    # Additional ALSA plugins
$ <b>sudo xbps-install pipewire</b>        # Low-latency audio/video router and processor
$ <b>sudo xbps-install alsa-pipewire</b>   # Pipewire ALSA configuration
$ <b>sudo xbps-install wiremix</b>         # PipeWire CLI UI
</pre>
</dd></dl>


8. [Optional] Improve battery usage with TLP - utility that basically does kernel settings
    tweaking that improve power consumption. More information about TLP
    [can be found here](https://linrunner.de/tlp/). More information about TLP-RDW (radio device wizard)
    [can be found here](https://linrunner.de/tlp/settings/rdw.html).

<dl><dd>
<pre>
$ <b>sudo xbps-install tlp tlp-ui</b>
$ <b>sudo ln -sv /etc/sv/tlp /var/service</b>
<div></div>
</pre>
</dd></dl>

9. [Optional] Install GTK themes and icons:

<dl><dd>
<pre>
$ <b>sudo xbps-install adwaita-icon-theme papirus-icon-theme</b>
</pre>
</dd></dl>

10. Setup XDG_RUNTIME

<dl><dd>
<pre>
$ <b>sudo xbps-install pam_rundir</b>
$ <b>sudo vim /etc/pam.d/login</b>
     # Append to end of file
     -session	optional	pam_rundir.so
     # Save ,Exit
$
</pre>
</dd></dl>

11. Setup Pipewire

<dl><dd><pre>
$ <b># mkdir -p /etc/pipewire/pipewire.conf.d</b>
$ <b># ln -s /usr/share/examples/wireplumber/10-wireplumber.conf /etc/pipewire/pipewire.conf.d/</b>
$ <b># mkdir -p /etc/pipewire/pipewire.conf.d</b>
$ <b># ln -s /usr/share/examples/pipewire/20-pipewire-pulse.conf /etc/pipewire/pipewire.conf.d/</b>
$ <b># mkdir -p /etc/alsa/conf.d</b>
$ <b># ln -s /usr/share/alsa/alsa.conf.d/50-pipewire.conf /etc/alsa/conf.d</b>
$ <b># ln -s /usr/share/alsa/alsa.conf.d/99-pipewire-default.conf /etc/alsa/conf.d</b>
</pre></dd></dl>

12. Reboot to finalize installation:

<dl><dd>
<pre>
$ <b>reboot</b>
</pre>
</dd></dl>




### [Optional] Dotfiles:
<dl><dd><pre>
$ git clone https://github.com/wtfxetra/dotfiles
$ cd dotfiles
$ ./install.sh
</pre></dd></dl>

### Start Niri
<dl><dd><pre>
$ dbus-run-session niri 
</pre></dd></dl>
