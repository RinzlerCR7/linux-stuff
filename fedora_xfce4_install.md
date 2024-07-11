Brother, I'll update the content. I'm new to Github so using your stuff as a reference.

![debian-xfce.jpg](debian-xfce.jpg)

## Fedora Xfce4 Install Guide

The standard Fedora installation process for Xfce desktop includes additional packages that may not be necessary for many users. This guide will allow you to install a minimal Xfce desktop, adding additional packages as needed.

## Requirements

* A Fedora netinstall LIVE USB.

* sudo privileges to install packages and run optional install script.

## ISO for installing Fedora

* [Download Fedora Everything net installer](https://alt.fedoraproject.org/)

## Installing Fedora with Xfce4

In the Software Selection menu, select `Xfce Desktop` as the base environment & add the following additional softwares for the selected environment,
* `Applications for the Xfce Desktop`
* `Multimedia support for Xfce`

_Note: You can skip these too maybe, installs a lot of crap._

## Switching from LightDM to SDDM

LightDM was randomly causing Black Screen during some of my system startups. If it happens with you too, switch to SDDM.

Remove LightDM:

```console
sudo dnf remove lightdm
```

Install SDDM:

```console
sudo dnf install sddm
```

After this, if SDDM doesn't automatically start on restart, use the following command.
_(Note: I don't remember if I've used this command. So it might be wrong.)_

```console
sudo systemctl start sddm
```

## Installing proprietary NVIDIA graphics card driver

Prerequisites:
* This guide requires the secure boot to be turned off to load up the unsigned NVIDIA kernel modules.

### Step #1: Update from the existing repositories

```console
sudo dnf update
```

### Step #2: Add the RPM Fusion nonfree repositories
Install nonfree repositories & enable fedora-cisco-openh264 repository:

```console
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
sudo dnf config-manager --enable fedora-cisco-openh264
```

### Step #3: Refresh to add the new repositories

```console
sudo dnf update --refresh
```

### Step #4: Install the driver & its dependencies
Update the system:

```console
sudo dnf update -y
```

_Note: Reboot if not on the latest kernel._

Install the following packages:

```console
sudo dnf install gcc
sudo dnf install automake
sudo dnf install akmod-nvidia
sudo dnf install xorg-x11-drv-nvidia-cuda
sudo dnf install xorg-x11-drv-nvidia-libs.i686
```

### Step #5: Wait for the kernel modules to load up
You must wait 5-10 minutes for the kernel modules to load. Please do not proceed to the next steps immediately.
_(Note: I'm not sure if it is necessary to wait. I've never tried not waiting. Just saw it in the doc, so putting it here.)_

Once the module is built,

```console
modinfo -F version nvidia
```

should outputs the version of the driver such as `440.64` & not `modinfo: ERROR: Module nvidia not found.`

### Step #6: Using KMS, add NVIDIA modeset argument to kernel boot parameters

```console
sudo grubby --update-kernel=ALL --args='nvidia-drm.modeset=1'
```

_Note: DO NOT restart after this step. Gives me black screen._

### Step #7: Install the following packages
Install `Vulkan`:

```console
sudo dnf install vulkan
```

Install `NVENC/NVDEC`:

```console
sudo dnf install xorg-x11-drv-nvidia-cuda-libs
```

_Note: xorg-x11-drv-nvidia-cuda already covers this installation._

Install `VDPAU/VAAPI`:

```console
sudo dnf install nvidia-vaapi-driver libva-utils vdpauinfo
```

### Step #8: Edit the X11 configuration
Install `xrandr`:

```console
sudo dnf install xrandr
```

Edit the `nvidia.conf` from `/usr/share/X11/xorg.conf.d/` to add

```console
Option "PrimaryGPU" "yes"
```

in the `OutputClass` section of it.

For example, use `MousePad` with root privileges to edit the `nvidia.conf` file. Use the following command to launch `MousePad` as root.

```console
sudo MousePad
```

Then open `nvidia.conf` file in `MousePad` from `/usr/share/X11/xorg.conf.d/` & make the required changes.

The file should look similar to this.

```console
#This file is provided by xorg-x11-drv-nvidia
#Do not edit

Section "OutputClass"
    Identifier "nvidia"
    MatchDriver "nvidia-drm"
    Driver "nvidia"
    Option "AllowEmptyInitialConfiguration"
    Option "SLI" "Auto"
    Option "BaseMosaic" "on"
    Option "PrimaryGPU" "yes"
EndSection

Section "ServerLayout"
    Identifier "layout"
    Option "AllowNVIDIAGPUScreens"
EndSection
```

You can see the additions in both sections.

Execute the following command to copy the display render details for the X11:

```console
sudo cp -p /usr/share/X11/xorg.conf.d/nvidia.conf /etc/X11/xorg.conf.d/nvidia.conf
```

Edit the `Xsetup` file from `/etc/sddm/` to add:

```console
xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
```

Restart SDDM:

```console
systemctl restart sddm
```

### Step #11: Reboot your system
Reboot your system and proceed to the next steps to verify the change in configuration.

```console
sudo reboot
```

### Step #12: Verify the configuration
Execute:

```console
glxinfo | egrep "OpenGL vendor|OpenGL renderer"
```

It should show your NVIDIA GPU.

Execute:

```console
neofetch
```

It should show your NVIDIA GPU under the GPU name.

Execute:

```console
glxgears
```

It should display 3D OpenGL graphics by running `glxgears` program.

## Switching between Nouveau/NVIDIA
To boot using the Nouveau driver instead of the NVIDIA binary driver, edit the kernel entry from grub bootloader & remove the following linux boot command arguments,

```console
rd.driver.blacklist=nouveau modprobe.blacklist=nouveau nvidia-drm.modeset=1
```

then boot with the updated boot command.

## Reference docs:
* [NVIDIA](https://rpmfusion.org/Howto/NVIDIA)
* [How to Set Nvidia as Primary GPU on Optimus-based Laptops](https://docs.fedoraproject.org/en-US/quick-docs/set-nvidia-as-primary-gpu-on-optimus-based-laptops/)
* [Configuration](https://rpmfusion.org/Configuration)
* [Optimus](https://rpmfusion.org/Howto/Optimus)
* [Third-Party Repositories](https://docs.fedoraproject.org/en-US/workstation-working-group/third-party-repos/)

# ARCHIVE

## Installing Debian without a desktop environment

As you progress through the debian installation, towards the end you will be presented with the following screen for Software selection:

![debian-installer.jpg](debian-installer.jpg)

Uncheck **Debian desktop environment** to install a minimal debian system.

## Update sources to testing or unstable (optional)

Update sources to `trixie`. The current testing branch.

`sudo nano /etc/apt/sources`:

```console
deb http://deb.debian.org/debian/ trixie main
#deb-src http://deb.debian.org/debian/ trixie main

deb http://security.debian.org/debian-security trixie-security main
#deb-src http://security.debian.org/debian-security trixie-security main

deb http://deb.debian.org/debian/ trixie-updates main
#deb-src http://deb.debian.org/debian/ trixie-updates main
```

Add `contrib non-free-firmware` after each `main` entry if you need special drivers or additional firmware.

The other option would be debian sid. Update `sources` as follows:

```console
deb http://deb.debian.org/debian/ unstable main
#deb-src http://deb.debian.org/debian/ unstable main
```

Upgrade your system:

```console
sudo apt update && apt upgrade
```

Reboot to load updated kernel and services.

## Quick install Xfce and required packages

```console
git clone https://github.com/coonrad/Debian-Xfce4-Minimal-Install.git
cd Debian-Xfce4-Minimal-Install
sudo ./xfce-install.sh
```

## Manually install Xfce and required packages

If you've read this far, and you're getting impatient:

```console
sudo apt install xfce4
```

This will give you a basic Xfce desktop with lightdm greeter. For additional plugins and ''goodies', add:

```bash
sudo apt install xfce4-goodies
```

This still may include too much 'stuff' for the minimalists among us. In that case, strip it down further and start with:

```bash
apt install \
    libxfce4ui-utils \
    thunar \
    xfce4-appfinder \
    xfce4-panel \
    xfce4-session \
    xfce4-settings \
    xfce4-terminal \
    xfconf \
    xfdesktop4 \
    xfwm4
```

From this point you should have a working Xfce desktop environment. You can reboot, and add only what you need going forward. If you plan to start xfce4 without lightdm, you may need to install xinit.

```bash
sudo apt install xinit
```

## Additional packages and configuration

Add a browser, password manager, document viewer, image viewer and office apps:

```bash
sudo apt install firefox-esr keepassxc atril ristretto libreoffice-gtk3 \
libreoffice-calc libreoffice-writer
```

Add NetworkManager and openvpn:

```bash
sudo apt install network-manager-openvpn network-manager-gnome \
network-manager-openvpn-gnome
```

Add a few nice icon themes to choose from:

```bash
sudo apt install paper-icon-theme moka-icon-theme papirus-icon-theme
```

Keep the default Adwaita theme as scientists have proven it is the best theme. Xfce comes with Lightdm for the display manager. It sources `.xessionrc` on login. Here are a few useful additions:

```bash
# source the system profile
# if [ -f /etc/profile ]; then
#     . /etc/profile
# fi

# QT5 qt5ct
export QT_QPA_PLATFORMTHEME=qt5ct

# QT5 scaling
# Uncomment for hidpi display
# export QT_AUTO_SCREEN_SCALE_FACTOR=1
# export QT_SCREEN_SCALE_FACTORS=2
```

Install `qt5ct` and `adwaita-qt` to have your QT apps match the default GTK theme.

```bash
sudo apt install qt5ct adwaita-qt
```

Launch `qt5ct` and configure the theme and fonts.

////////////////////////////////////////////////
### Step #2: Add the RPM Fusion repository for NVIDIA drivers
Execute:
```bash
sudo dnf config-manager --set-enable rpmfusion-nonfree-nvidia-driver
```

```bash
sudo dnf install gcc kernel-headers kernel-devel akmod-nvidia xorg-x11-drv-nvidia xorg-x11-drv-nvidia-libs xorg-x11-drv-nvidia-libs.i686
```

### Step #6: Read from the updated kernel modules
Execute:
```bash
sudo akmods --force
sudo dracut --force
```

_Note: DO NOT restart after this step. Gives me black screen._
