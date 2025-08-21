# 0. Introduction/Setup.
This tutorial is about how to install Arch Linux. As of current, the tutorial will result in a minimal install of Arch with just enough functionality to successfully boot and be able to install new packages. In essence, the bare minimum working Arch Linux system. I used the [Arch Wiki](https://wiki.archlinux.org/title/Installation_guide) as a reference for installing Arch Linux. Another thing to note is that I used an Oracle VirtualBox Virtual Machine environment. On real hardware, you will likely need to install additional drivers for your wifi adapter and whatever other peripheral devices you have.

# 1. VirtualBox setup.
Firstly, download the Arch Linux ISO from one of the mirrors listed on their [website](https://archlinux.org/), via HTTP or torrenting. Then, create a new Virtual Machine using the Arch Linux installer ISO. The most important setting to enable is EFI. This can be found under the motherboard settings. I gave my machine 2 processors, 4924MB of memory, 12GB of storage (definitely overkill), and 128MB of video memory (also probably overkill). I left everything else to the defaults. It's definitely possible to give an Arch Linux VM far less resources than this, so feel free to give it as many or as few resources as you see fit. However, it is worth checking the Arch Linux website for its minimum recommended specs before going too crazy with it.

# 2. Install Arch Linux.
Boot up the VM, you will find yourself logged in as root with nothing more than a CLI (command line interface). Run the following commands to install Arch Linux.

> cat /sys/firmware/efi/fw_platform_size

Should be 64. If it says `No such file or directory` then it probably isn't in UEFI mode which is an issue. If this happens, double check your motherboard settings (under the System tab) and ensure that the `Enable EFI (special OSes only)` checkbox is checked.

Run the following commands to make sure your network connection is working. In a VM, you have an "ethernet" connection and everything should just work out of the box.

> ip link

> ping ping.archlinux.org

Run the following command to see your disks and filesystems. Identify the VM's disk, it will show up as an unformatted drive with whatever size you specified in the VM's settings. For me, it was `/dev/sda`.

> lsblk

Run the following command to enter a disk partitioning program. Instructions are provided below for creating a /boot partition and a root (/) partition.

> fdisk /dev/sdX

For /boot:
`n` `p` enter, enter, `+1G` or `+500M`

For /:
`n` `p` enter enter enter

`w`

Note that I opted not to create a swap partition, as I believe modern computers have enough RAM that it's typically unnecessary. On a machine with a small amount of RAM, I.E. 4GB or less, or even 8GB or less, a swap partition could be justified but arguably a swapfile is better since it's easy to resize. This way, it's far more flexible as resizing partitions on a machine that has already been installed is a pain. This is just my preference, and the point of running Arch is to do things however you want so feel free to create a swap partition if you wish. Instructions for doing so can be found on the Arch Wiki installation guide.

Run the following command to ensure the partitions were successfully created, and to see what labels were assigned to them. (e.g. sda1, sda2)

> lsblk

Run the following command to format the root partition (/) as an ext4 filesystem. For me it was sda2, and was 11GB. In any case it should be the largest partition unless you opted for a radically different disk setup, in which case you likely already know what you're doing. Hopefully.

> mkfs.ext4 /dev/sdXX

It's important that the /boot partition is formatted as FAT32. Run the following command to do that. For me the label was sda1.

> mkfs.fat -F 32 /dev/sdXX

Mount both disks, the root partition disk first as /mnt, and the boot partition as /mnt/boot. If you have other partitions, mount or set them as swap appropriately depending on what they are. If you're strictly following this tutorial you won't have any other partitions to worry about.

> mount /dev/sdXX /mnt

> mount --mkdir /dev/sdXX /mnt/boot

Run the following command to use pacman (Arch's package manager) to install essential packages to the system. Since I have an AMD CPU, I install the `amd-ucode` package. If you have an intel CPU, you will want the `intel-ucode` package instead. The ucode package may not be absolutely essential, but I would recommend it for security reasons. It can also fix various hardware bugs so overall it makes one's system more stable and functional. I also opted for vim as my text editor, you may use nano or something else if you don't like vim or don't know how to use it. I also chose `networkmanager` as my network manager. You may use a different one but it is essential that you have one and it works otherwise you won't be able to install new packages or update packages. 

> pacstrap -K /mnt base linux linux-firmware amd-ucode vim networkmanager

If the command fails saying a mirror took too long to respond, try running it again.

Now, we need to generate the /etc/fstab file. If you are unaware, this file tells Linux about all of the partitions and disks on your system that need to be mounted when your system boots up. In our case it will just be the root and boot partitions. Run the following command to generate the file.

> genfstab -U /mnt >> /mnt/etc/fstab

Run the following command to inspect the newly generated fstab file. Make sure both partitions are there and everything appears correct.

> cat /mnt/etc/fstab

Now, we will run the following command in order to start a shell session inside the Arch Linux system itself. Up until now, we have been in a shell session in the Arch Linux live installer.

> arch-chroot /mnt

Run the following command to set the timezone. Mine was US Pacific time, yours may be different. You can run `ls /usr/share/zoneinfo` to see what timezones are available and select yours accordingly.

> ln -sf /usr/share/zoneinfo/US/Pacific /etc/localtime

The following command may not be necessary for a VM, but on real hardware this would sync your motherboard's hardware clock with your Arch system's time. Essentially this ensures your computer is keeping time accurately.

> hwclock --systohc

Run the following command to generate the locale.gen file you will be editing shortly.

> locale-gen

Run the following command to edit the locale file, and uncomment `en_US.UTF-8 UTF-8` and `en_US ISO-8859-1`. If you aren't an American, you might want to instead uncomment whichever locales correspond to your country/region. If you installed something other than vim, replace vim with that (e.g. nano).

> vim /etc/locale.gen

Run the following command to create the locale.conf file, and write `LANG=en_US.UTF-8` as the content. Once again, if you aren't an American you may want to set this to whatever corresponds to your country/region. If you installed something other than vim, replace vim with that (e.g. nano).

> vim /etc/locale.conf

Run the following command again so that the above changes take effect.

> locale-gen

Run the following command to create a hostname file. This will be a human-readable string which serves as a name for your computer. You may set it to anything you like, so long as it's 1 to 63 characters, only lowercase letters, numbers, and dashes, and it doesn't start with a dash. If you installed something other than vim, replace vim with that (e.g. nano).

> vim /etc/hostname

Run the following command to set the root password for your Arch install's root user. You will be asked to enter a password of your choice twice. Be sure to remember it or write it down somewhere so you don't lose it.

> passwd

Now, we will install our boot loader. I opted for grub, but you may install whatever bootloader you would like. To install grub's packages, run the following command.

> pacman -S grub efibootmgr

Grub needs to be written to the /boot partition now. Run the following command to do that.

> grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB

Finally, grub needs a configuration file generated so that it knows where all of your kernels are. Run the following command to do that.

> grub-mkconfig -o /boot/grub/grub.cfg

Exit the shell session, this will take you back to the Arch installer.

> exit

Run the following command to unmount the newly installed Arch system. If it says the disk is busy and can't be unmounted, wait a second or two and try again.

> umount -R /mnt

Finally, reboot!

> reboot

# 3. First boot.
Log into your root account using whatever password you set before. If you try doing anything that requires a network connection, you might notice that you don't have a working internet connection. This is a problem you can fix by running the following two commands:

> systemctl enable NetworkManager.service

> systemctl start NetworkManager.service

Now, your internet should be working. If you installed a different network manager, the procedure for enabling it might be different. At this point, you should have a working Arch Linux system. Any additional packages you need are installable using pacman, and you are officially an Arch Linux user. Congratulations.
