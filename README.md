
# Arch Installation Guide

## 1. Internet and mirrors
1. Connect to wifi
```bash
iwctl
station DEVICE scan
station DEVICE get-networks
station DEVICE connect SSID
```
2. Update mirrorlist:
```bash
reflector --country Poland --latest 5 --sort rate --save /etc/pacman.d/mirrorlist
```

## 2. Partition disk
1. Disk layout
```bash
lsblk /dev/sda # Verify disk and partitions

gdisk /dev/sda # GPT partitioning
``````

2. `BOOT` partition
    1. Type `n`
    2. Enter to select first sector
    3. `+512M` to allocate space
    4. Type `c` and then write partition name e.g. `BOOT` and hit Enter

3. `SWAP` partition
    1. Type `n`
    2. Enter to select first available sector
    3. `+8G` to allocate space
    4. Type `c` and then write partition name e.g. `SWAP` and hit Enter

4. `ROOT` partition
    1. Type `n`
    2. Enter to select first available sector
    3. Hit enter again to select rest of avalable space
    4. Type `c` and then write partition name e.g. `ROOT` and hit Enter

5. After you\'re done type `w` to write changes to the disk
6. Exit program
7. Formatting
```bash
mkfs.fat -F32 /dev/sda1
mkswap /dev/sda2
mkfs.ext4 /dev/sda3
```

3. Mounting
```bash
mount /dev/sda3 /mnt
mount --mkdir /dev/sda1 /mnt/boot
swapon /dev/sda2
```

## 3. Installation

```bash
pacstap -K /mnt base base-devel git linux linux-firmware vim openssh reflector rsync zsh
```

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

## 4. Chroot setup

```bash
arch-chroot /mnt
ln -sf /usr/share/zoneinfo/Europe/Warsaw /etc/localtime
hwclock --systohc --utc
reflector --country Poland --latest 5 --sort rate --save /etc/pacman.d/mirrorlist

vim /etc/locale.gen # uncomment your locale
locale-gen
echo LANG=pl_PL.UTF-8 > /etc/locale.conf
echo arch-thinkpad > /etc/hostname

passwd # set root password

# create user
useradd -m -G sys,log,network,floppy,scanner,power,rfkill,users,video,storage,optical,lp,audio,wheel,adm -s /bin/zsh sdrozdz
passwd sdrozdz
id sdrozdz # check groups

# uncomment wheel section to enable sudo
EDITOR=vim visudo
```

## 5. VirtualBox Setup (optional)

```bash
# Guest Editions and allow video resizing
# Info: https://wiki.archlinux.org/index.php/VirtualBox
pacman -S --noconfirm linux-headers virtualbox-guest-utils virtualbox-host-modules-arch nfs-utils
echo -e 'vboxguest\nvboxsf\nvboxvideo' > /etc/modules-load.d/virtualbox.conf

systemctl enable vboxservice
systemctl enable rpcbind
```

## 6. Grub setup
```bash
pacman -S grub os-prober efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ArchLinux
grub-mkconfig -o /boot/grub/grub.cfg
mkinitcpio -p linux
```

## 7. Install packages

If you want to use paclist then use command:
```bash
cd paclist
pacman -S --needed - < Base.txt
pacman -S --needed - < Drivers.txt
pacman -S --needed - < Net.txt
pacman -S --needed - < Fonts.txt
pacman -S --needed - < Media.txt
pacman -S --needed - < Xorg.txt
pacman -S --needed - < Xfce.txt
pacman -S --needed - < Apps.txt
```

## 8. Enable services

```bash
systemctl enable avahi-daemon
systemctl enable bluetooth
systemctl enable haveged.service
systemctl enable cups.service
systemctl enable firewalld.service
systemctl enable fstrim.service
systemctl enable sddm.service
systemctl enable NetworkManager
systemctl enable reflector.timer
systemctl enable sshd
systemctl enable upower
```

## 9. Exit and reboot

```bash
exit
umount -a
reboot
```