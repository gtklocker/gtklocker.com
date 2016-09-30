---
layout: post
title:  "Running Arch Linux as a VMware guest"
date:   2016-09-30 13:12:04 +0300
---

# Why?
My native OS is macOS and I need a Linux environment to do development on. Arch Linux is the most sensible choice for a developer these days since it has the latest versions of every package and AUR.

Since installing Arch as VMware guest has many quirks, I'm writing this post to help others and to help myself stay sane in case I do this procedure again. This [wiki entry][wikiguest] covers most of the stuff in this post.

So, here goes.

# Basic installation
Create a VM and use the [installation guide][wikibeginners] in the Arch Wiki to set up Arch. Make sure to install GRUB. Reboot.

# open-vm-tools
We're going to use the opensource `open-vm-tools` instead of the normal VMware Tools (which are soon to be deprecated).

    pacman -S open-vm-tools
    cat /proc/version > /etc/arch-release # open-vm-tools needs this

Now let's enable the required services

    systemctl enable vmware-vmblock-fuse
    systemctl enable vmtoolsd

We need to install `gtkmm` for drag & drop and copy/paste with the host to work:

    pacman -S gtkmm

# Xorg
Install Xorg normally. We only need some extra drivers for the virtual mouse and graphics.

    pacman -S xorg-server
    pacman -S xorg-input-vmmouse xf86-video-vmware mesa

# Desktop Environment
From here on it's totally your choice. You can use i3, Gnome, or any DE or WM you want. The Arch Wiki has you covered.

I'm going to show the installation of Deepin, since this is what I used for my guest. It's as easy as

    pacman -S deepin deepin-extra

Now let's install our display manager. This display manager will act as our login screen.

    pacman -S lightdm

Let's use the Deepin greeter for lightdm now. Just edit the file `/etc/lightdm/lightdm.conf`, find the line that starts with `greeter-session=` and change it to `greeter-session=lightdm-deepin-greeter`.

To enable lightdm, do

    systemctl enable lightdm

# Done!
From here on, it should be smooth sailing. Enjoy! 😎

![Screenshot of guest]({{ site.url }}/assets/archlinux-vmware-guest/done.png)

Below you'll find some problems you may (or may not) encounter and how to solve them.

# Potential issues

#### Jiggly mouse
If your mouse jumps or is unresponsive, you may need to modify your .vmx file. My .vmx is at `~/Documents/Virtual Machines.localized/Arch Linux.vmwarevm/Arch Linux.vmx` on macOS. Find your .vmx and append these lines, with the VM shut down.

    mouse.vusb.enable = "TRUE"
    mouse.vusb.useBasicMouse = "FALSE"
    usb.generic.allowHID = "TRUE"

Then start up the VM again and the issues should be gone.

#### Can't boot
Sometimes this message may appear and freeze the boot process.

    intel_rapl: no valid rapl domains found in package 0

This can be fixed by enabling the virtual extensions on the VM.

![Virtual extensions dialog]({{ site.url }}/assets/archlinux-vmware-guest/vt-x.png)

[wikibeginners]: https://wiki.archlinux.org/index.php?title=Installation_guide
[wikiguest]: https://wiki.archlinux.org/index.php/VMware/Installing_Arch_as_a_guest
