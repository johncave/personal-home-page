+++
date = '2025-03-24T17:30:36+13:00'
draft = true
title = 'My Arch Linux Initial Setup'
+++

# Arch Linux Initial Setup
I wanted to document how I install and do initial set up for [Arch Linux](https://archlinux.org/) on my machines. Both for myself and in case anyone else finds it useful. 

Arch has a reputation for being a complicated manual process to install, but it's actually had a command line installer for some time now. This guide will assume you're familiar with Linux and just want to get quickly to a good basic setup.

## Download and Burn the ISO
As usual with any Linux distro, the first thing to do is [Download Arch](https://archlinux.org/download/) ISO file and burn it to a USB. I personally use [Balena Etcher](https://etcher.balena.io/) to do that burning. 

## Boot the ISO 
> On most systems, pressing the ESC key during initial boot will show the list of keys to access UEFI functions. So there's no need to guess. 

First, go into the BIOS of the system and disable secure boot to enable booting the Arch install medium. Then, reboot the system and select the boot devices option in the BIOS to select the USB drive. 

## Connect to WiFi (optional)
It's easiest to plug the device into a wired network at this point, but for devices where that isn't an option I'll record here how to connect to WiFi with the command line.

First, enter the `iwctl` command line, by typing:
```
iwctl
```
This will put you at the iwctl prompt. Type the following:
```
station list # Shows available network cards, most likely you will use wlan0
station wlan0 scan
station wlan0 get-networks # Confirm you can see your network 
```
Finally, connect to the network with the command
```
# Tab completion is available for the network name
station wlan0 connect "My Wifi"
station wlan0 show # Show the status of the network connection
quit
```
You can then check the network status with `ping google.com` if desired. 

## Install Arch with Archinstall 

Archinstall should be familiar if you've installed Linux before. Here's some notes about my setup.
- Use BTRFS subvolumes with default structure as the partitioning layout, to support Timeshift later.
- Recommend GRUB bootloader.
- Enable Unified Kernel Images to enable Secure Boot later.
- Network Configuration must be set to Network Manager to enable desktop configuration.
- I like to add `firefox` to additional packages so there is a web browser ready to use on first boot. 
- Enable the Multilib repo to install Steam.
- Use Profiles to select the desktop you want. I usually go with KDE Plasma, but on touchscreen laptops or tablets I'd recommend GNOME.

When `archinstall` is finished running, use `no` to skip chrooting into the new installation. Then type `reboot` to reboot into the new install. Remember to remove the arch install USB disk once the system has shut down.

## Initial Configuration
Log in to the new configuration and do initial click-around configuration. The KDE welcome screen will take you through most of this. 
- Connect to WiFi
- Change pointer speed and scroll direction. 
- Set dark theme(!). Or, my preferred them on KDE, 
- On KDE, disable session restore in `System Settings > Session`

## Install AUR Helper
AUR helper helps you get software from the excellent Arch User Repositories. I recommend installing one right away. 
```
sudo pacman -S --needed base-devel git
git clone https://aur.archlinux.org/paru-bin.git
cd paru-bin
makepkg -si
cd ..
rm -rf paru-bin
```
Follow the prompts and install paru onto your system. Alternatively you could substitute `paru` for `yay` or your favourite helper. 

## Install Packages
This installs a whole bunch of packages that make up a working system for me. 
```
paru -Syu librewolf-bin fish keychain visual-studio-code-bin timeshift-autosnap nano btop magic-wormhole 
```

## Configure Timeshift Autosnap
The `timeshift-autosnap` function will create a BTRFS snapshot of the `/` partition almost instantly whenever the system is updated with `sudo pacman -Syu`. I recommend editing `/etc/timeshift-autosnap.conf` and updating `maxSnapshots=12` as I find 3 can be too few sometimes. 

If you installed Arch with the Grub bootloader and the `grub-btrfs` package, timeshift can make boot entries to take you back in time too. I've never found this necessary, as Arch already has a fallback boot option by default.  

## Configure Fish and SSH Keychain
I prefer to change my shell to `fish` but it's worth noting that this breaks using VSC remote on the machine. It can always be changed back temporarily to use VSC remote. 

### Copy ssh files
First, copy needed SSH keys from an existing machine using `magic-wormhole`:
``` 
wormhole send ~/.ssh
```
Warning! This will copy things like SSH config and authorized_keys as well.

On the new machine, run the command shown by magic-wormhole to receive the files. Make sure to run it in the home directory. 
```
cd ~
# Example
wormhole receive 7-monkey-bacon
```

### Do initial run of fish and configure keychain
At the terminal, run `fish` to start fish and create config directories. Then edit the config file to enable keychain on fish startup.
```
nano ~/.config/fish/conf.d/keychain.fish
```
Add the following content:
```
if status is-interactive
    # Add desired keys on following line
    keychain --quiet --eval ~/.ssh/id_ed25519 ~/.ssh/id_github | source
end
```
Type `exit` and then `fish` again to confirm that you're prompted for the SSH key's password if enabled. The keychain SSH variables will be injected into every running fish session, and you won't be prompted for the password again until after reboot. 

### Change Default Shell
I change my default shell to fish.
```
chsh
```
Enter your password, and change the shell to `/usr/bin/fish`.

## To Do
Is a couple more things I'd like to perfect about this process
- Enable secure boot 
- Enable TPM unlock of the boot drive
- Configure gestures with touchegg.
- Add instructions for keychain on BASH

## That's All Folks
You've now installed Arch and got it to the point of being able to develop. Some of the steps are quirky to my personal setup, but you're welcome to skip those ones. 