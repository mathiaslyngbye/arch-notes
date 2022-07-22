# Arch Linux Installation Notes
This describes the process and notes I have gathered from installing Arch Linux.

## Installation
### Preparation
Force the desired boot mode (likely UEFI) from bios.
Boot into the installation media.
This should provide you with an Arch Linux shell.

#### Verify boot mode
```bash
ls /sys/firmware/efi/efivars
```
If file exists, you are in UEFI.

#### Networking during installation
I've run with a wired connection. However, wireless may be established.
Connection may be verified through the following commands.
```bash
ip addr show
ping -c 5 8.8.8.8
```

#### Keyboard layout during installation
Quality of life for installation only. I'm danish so I load danish keys.
```bash
loadkeys dk-latin1
```

#### Time and timezone during installation
This is just to make sure. This might need to be repeated when done.
```bash
timedatectl set-ntp true
timedatectl set-timezone "Europe/Copenhagen"
timedatactl status
```

### Setup disk
I use UEFI without encryption.

#### Partition and format disk
Note, in all cases I already had 512M EFI (sdc1) and rest Linux filesystem (sdc2).
Verify your drive location (e.g. via ```fdisk -l``` and partition as desired. Consult wiki if in doubt.
The following commands formats my existing partitions.
```bash
mkfs.fat /dev/sdc1
mkfs.ext4 /dev/sdc2
```
#### Mounting drive
Mount the newly formatted drive.
```bash
mount /dev/sda2 /mnt
```

### Installing Arch

#### Install (Arch) Linux kernel and firmware
Pacstrap allows pacman install before chroot. After chroot use ```pacman -S```.

Install arch base and change root.
```bash
pacstrap /mnt base
arch-chroot /mnt
```

Install linux kernels (and corresponding headers) as desired. I use ```linux-lts```.
Remember this when installing nvidia drivers later.
```bash
linux-lts linux-firmware linux-lts-headers
```

Generate filesystem tables.
```bash
genfstab -U -p /mnt >> /mnt/etc/fstab
```

### Installing optional things

#### Vim
```bash
pacman -S vim
```

#### Basic networking
```bash
pacman -S base-devel
pacman -S networkmanager wpa_supplicant wireless_tools netctl
systemctl enable NetworkManager
```

#### Locale
Edit ```/etc/locale.gen``` and uncomment ```en_US.UTF-8 UTF-8``` and other needed locales. 
Generate the locales by running the following.
```bash
locale-gen
vim /etc/locale.conf // add LANG=en_US.UTF-8
```

#### Sudo
Add root user and personal user.
```bash
passwd // root
useradd -m -g users -G wheel mal
passwd mal
```

Add sudo.
```bash
pacman -S sudo
EDITOR=vim visudo
```
Uncomment line ```%wheel ALL=(ALL) ALL```

### Grub

#### Install bootloader
```bash
pacman -S grub efibootmgr dosfstools os-prober mtools
mkdir /boot/EFI
mount /dev/sda1 /boot/EFI
grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck
```
#### Add locale
Verify if ```/boot/grub/locale``` exists. If not, run the following.
```bash
cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo
```

#### Find Windows
Edit ```/etc/default/grub``` and uncomment the following line.
```bash
GRUB_DISABLE_OS_PROBER=false
```

Mount windows for os-prober to find.
```bash
mount /dev/<windows partition> /mnt/<wherever>
```

Update grub config.
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

### Post installation
#### Swap
Login as root.
```bash
su
cd root/
```
Generate swap file
```bash
dd if=/dev/zero of=/swapfile bs=1M count=2048 status=progress
chmod 600 /swapfile
mkswap /swapfile
```

Enable swap - don't fuck this line up.
```bash
echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab¨
mount -a
swapon -a
```

Verify
```bash
cat /etc/fstab
free -m
```

### Post installation
#### Time and timezone
This is just to make sure. This might need to be repeated when done.
```bash
timedatectl set-ntp true
timedatectl set-timezone "Europe/Copenhagen"
timedatectl status
systemctl enable systemd-timesyncd
```


#### Hostname
Set hostname:
```bash
hostnamectl set-hostname <hostname>
```
Add to ```/etc/hosts```:
```bash
127.0.0.1 localhost
127.0.1.1 <hostname>
```

#### Microcode for CPU
```bash
pacman -S intel-ucode
```


### Graphics

#### xorg
```bash
pacman -S xorg-server
```

install nvidia driver corresponding to kernel (i.e. ```linux``` -> ```nvidia```, ```linux-lts``` -> ```nvidia-lts```)
Generate xorg.conf with nvidia-settings, or copy below.
```bash
pacman -S nvidia-lts nvidia-settings
```

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

### Desktop environment
```bash
pacman -S mate mate-extra
```

Login manager
```bash
pacman -S lightdm lightdm-gtk-greeter
systemctl enable lightdm
```

### Audio
Install alsa and pulseaudio-alsa.



