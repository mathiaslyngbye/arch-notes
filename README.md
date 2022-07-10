# Arch Linux Installation Notes
This describes the process and notes I have gathered from installing Arch Linux.

## Installation
### Setup
Force the desired boot mode (likely UEFI) from bios.
Boot into the installation media.
This should provide you with an Arch Linux shell.

#### Verify boot mode
```bash
ls /sys/firmware/efi/efivars
```
If file exists, you are in UEFI.

#### Verify internet connection
I've run with a wired connection. However, wireless may be established.
Connection may be verified through the following commands.
```bash
ip addr show
ping -c 8.8.8.8
```

#### Set keyboard layout
Quality of life for installation only. I'm danish so I load danish keys.
```bash
loadkeys dk-latin1
```

#### Set time and timezone
This is just to make sure. This might need to be repeated when done.
```bash
timedatectl set-ntp true
timedatectl set-timezone "Europe/Copenhagen"
timedatactl status
```

#### Partition and format disk
Note, in all cases I already had 512M EFI (sda1) and rest Linux filesystem (sda2).
Verify your drive location (e.g. via ```fdisk -l``` and partition as desired. Consult wiki if in doubt.
The following commands formats my existing paritions.
```bash
mkfs.fat /dev/sda1
mkfs.ext4 /dev/sda2
```

### Installing Arch

#### Mounting drive
Mount the newly formatted drive.
```bash
mount /dev/sda2 /mnt
```

#### Install (Arch) Linux kernel and firmware
Note: pacstrap allows pacman install before chroot. After chroot use pacman -S.
```bash
pacstrap /mnt base linux linux-firmware linux-headers // This installs headers | ALTERNATIVE: install linux-lts-xxxx
genfstab -U -p /mnt >> /mnt/etc/fstab
arch-chroot /mnt
```


### Installing nice stuff
```bash
pacman -S base-devel
pacman -S networkmanager wpa_supplicant wireless_tools netctl
systemctl enable NetworkManager
```

#### Locale
Edit /etc/locale.gen and uncomment en_US.UTF-8 UTF-8 and other needed locales. Generate the locales by running: 
```bash
locale-gen
vim /etc/locale.conf // add LANG=en_US.UTF-8
```

#### Root and user
```bash
passwd // root
useradd -m -g users -G wheel mal
passwd mal
```

### Grub and stuff
```bash
pacman -S grub efibootmgr dosfstools os-prober mtools
mkdir /boot/EFI
mount /dev/sda1 /boot/EFI
grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck
```
There is more... remember to add it.

### Audio
Install alsa and pulseaudio-alsa.

### Graphics
install nvidia driver corresponding to kernel (i.e. linux -> nvidia, linux-lts -> nvidia-lts)
Generate xorg.conf with nvidia-settings, or copy below.

Add to mkinitcpio:
Edit the following line of /etc/mkinitcpio.conf
```
MODULES=(i915? nouveau? vboxvideo? vmwgfx?)
```
Remove nouveau and add nvidia nvidia_modeset nvidia_uvm nvidia_drm

Then run
```
sudo mkinitcpio -p linux
```

Add hook to pacman (see https://wiki.archlinux.org/title/NVIDIA#mkinitcpio)
```
[Trigger]
Operation=Install
Operation=Upgrade
Operation=Remove
Type=Package
Target=nvidia
Target=linux
# Change the linux part above and in the Exec line if a different kernel is used

[Action]
Description=Update NVIDIA module in initcpio
Depends=mkinitcpio
When=PostTransaction
NeedsTargets
Exec=/bin/sh -c 'while read -r trg; do case $trg in linux) exit 0; esac; done; /usr/bin/mkinitcpio -P'
```

