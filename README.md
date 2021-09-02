# Arch Linux Installation Notes
This describes the process and notes I have gathered from installing Arch Linux.

## Installation
### Booting into installation media
Force the desired boot mode (likely UEFI) from bios.
Boot into the installation media.
This should provide you with an Arch Linux shell.

#### Verify boot mode
ls /sys/firmware/efi/efivars // If exists, you are in UEFI

// Verify internet
ip addr show
ping -c 8.8.8.8

// Keyboard
loadkeys dk-latin1

// Set time
timedatectl set-ntp true
timedatectl set-timezone "Europe/Copenhagen"
timedatactl status // <= verify

// Set up disk
I ALREADY HAD 512M EFI (sda1) and rest Linux filesystem (sda2)
mkfs.fat /dev/sda1
mkfs.ext4 /dev/sda2
mount /dev/sda2 /mnt

// System
// Note: pacstrap allows pacman install before chroot. After chroot use pacman -S.
pacstrap /mnt base linux linux-firmware linux-headers // This installs headers | ALTERNATIVE: install linux-lts-xxxx
genfstab -U -p /mnt >> /mnt/etc/fstab
arch-chroot /mnt


// Installing nice stuff
pacman -S base-devel
pacman -S networkmanager wpa_supplicant wireless_tools netctl

// Locale
Edit /etc/locale.gen and uncomment en_US.UTF-8 UTF-8 and other needed locales. Generate the locales by running: 
locale-gen
vim /etc/locale.conf // add LANG=en_US.UTF-8

// Root and user
passwd // root
useradd -m -g users -G wjeel mal
passwd mal


================================
// Grub and stuff
pacman -S grub efibootmgr dosfstools os-prober mtools
mkdir /boot/EFI

