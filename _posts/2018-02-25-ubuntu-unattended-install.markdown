---
layout: single
title: "Ubuntu - unattended install"
description: Ubuntu unattended install
date:   2018-02-25 23:33:34 -0000
categories: linux cryptocurrency
---

If you are working on a project that requires setting up linux on multiple machines, then investing time in a remastered ISO for an unattended install is worthwhile, because it takes the pain out of taking repetitive steps.

[Here](https://github.com/ki1cx/ubuntu-unattended) is the link to repo.

Preseeding allows you to create an unattended install ISO. In my example, I've remastered an Ubuntu ISO, but the concept is similar with other flavors of linux. Here is the preseed that I've put together for building my machines.

{% raw %}
```
# regional setting
d-i debian-installer/language                               string      en_US:en
d-i debian-installer/country                                string      US
d-i debian-installer/locale                                 string      en_US
d-i debian-installer/splash                                 boolean     false
d-i localechooser/supported-locales                         multiselect en_US.UTF-8
d-i pkgsel/install-language-support                         boolean     true

# keyboard selection
d-i console-setup/ask_detect                                boolean     false
d-i keyboard-configuration/modelcode                        string      pc105
d-i keyboard-configuration/layoutcode                       string      us
d-i keyboard-configuration/variantcode                      string      intl
d-i keyboard-configuration/xkb-keymap                       select      us(intl)
d-i debconf/language                                        string      en_US:en

# network settings
d-i netcfg/choose_interface                                 select      auto
d-i netcfg/dhcp_timeout                                     string      5
d-i netcfg/get_hostname                                     string      {{hostname}}
d-i netcfg/get_domain                                       string      {{hostname}}

# mirror settings
d-i mirror/country                                          string      manual
d-i mirror/http/hostname                                    string      archive.ubuntu.com
d-i mirror/http/directory                                   string      /ubuntu
d-i mirror/http/proxy                                       string

# clock and timezone settings
d-i time/zone                                               string      {{timezone}}
d-i clock-setup/utc                                         boolean     false
d-i clock-setup/ntp                                         boolean     true

# user account setup
d-i passwd/root-login                                       boolean     false
d-i passwd/make-user                                        boolean     true
d-i passwd/user-fullname                                    string      {{username}}
d-i passwd/username                                         string      {{username}}
d-i passwd/user-password-crypted                            password    {{pwhash}}
d-i passwd/user-uid                                         string
d-i user-setup/allow-password-weak                          boolean     false
d-i passwd/user-default-groups                              string      adm cdrom dialout lpadmin plugdev sambashare
d-i user-setup/encrypt-home                                 boolean     false

# configure apt
d-i apt-setup/restricted                                    boolean     true
d-i apt-setup/universe                                      boolean     true
d-i apt-setup/backports                                     boolean     true
d-i apt-setup/services-select                               multiselect security
d-i apt-setup/security_host                                 string      security.ubuntu.com
d-i apt-setup/security_path                                 string      /ubuntu
tasksel tasksel/first                                       multiselect Basic Ubuntu server
d-i pkgsel/upgrade                                          select      safe-upgrade
d-i pkgsel/update-policy                                    select      none
d-i pkgsel/updatedb                                         boolean     true

# disk partitioning - unmount
d-i preseed/early_command                                   string      umount /media || true

# disk partitioning
d-i partman-auto/disk                                       string      /dev/sda
d-i partman-auto/method                                     string      lvm
d-i partman-lvm/device_remove_lvm                           boolean     true
d-i partman-lvm/confirm                                     boolean     true
d-i partman-lvm/confirm_nooverwrite                         boolean     true
d-i partman-auto-lvm/guided_size                            string      max
d-i partman-auto/choose_recipe                              select      atomic
d-i partman-partitioning/confirm_write_new_label            boolean     true
d-i partman/choose_partition                                select      finish
d-i partman/confirm                                         boolean     true
d-i partman/confirm_nooverwrite                             boolean     true

# grub boot loader
d-i grub-installer/only_debian                              boolean     true
d-i grub-installer/with_other_os                            boolean     true
d-i grub-installer/bootdev                                  string      /dev/sda

# finish installation
d-i finish-install/reboot_in_progress                       note
d-i finish-install/keep-consoles                            boolean     false
d-i cdrom-detect/eject                                      boolean     true
d-i debian-installer/exit/halt                              boolean     false
d-i debian-installer/exit/poweroff                          boolean     true

```
{% endraw %}

The included _create-unattended-iso.sh_ script will ask for user input and fill in the following attributes for you.
 
* timezone
* username
* password
* ssh port

There were 4 pain points in getting the whole process to work smoothly.

1. Automating the partitioning of the disk. 

    Reading this [guide](https://help.ubuntu.com/16.04/installation-guide/i386/apbs04.html), specifically the _Partitioning example_ section helped. In the example, including the following lines ensures that no user confirmation is required to move on to the next steps.

    ```
    # This makes partman automatically partition without confirmation, provided
    # that you told it what to do using one of the methods above.
    d-i partman-partitioning/confirm_write_new_label boolean true
    d-i partman/choose_partition select finish
    d-i partman/confirm boolean true
    d-i partman/confirm_nooverwrite boolean true
    ```

2. Target installation disk already has mounted partitions. 

    This was easily solved by adding the following in the preseed file.[^1]

    ```
    d-i preseed/early_command string umount /media || true
    ```
   
3. Run custom commands to install additional packages after installation.

    This can be done using the _late_command_ preseed syntax.[^3]

    ```
    d-i preseed/late_command string <your commands>
    ```
    
4. Burn image to USB.

    Once you have your preseed file and remastered the ISO, then you can burn the image to a USB as follows.[^2]

    ```
    dd if=<remastered iso image> of=<usb device> bs=4M && sync
    ```
    
It is important to consider whether you want the server to simply reboot/halt/powered off. I chose to turn off the power all together to signify that the installation is complete. Allowing it to reboot while the USB drive is connected will simply re-install the system again.

#### References

[^1]: http://matelakat.blogspot.com/2014/05/ubuntu-installer-unmount-partitions.html
[^2]: https://unix.stackexchange.com/questions/323238/how-to-make-a-bootable-usb-from-commandline
[^3]: https://askubuntu.com/questions/667336/preseed-late-command-ubuntu-server-15-04

