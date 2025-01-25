# ArchLinux/ArtixLinux(dinit) basic installation
### NB.The guide is incomplete, you can follow up to the reboot, then you are on your own
Login as root (Artix) 
```
sudo su
```

### Partioning (Arch/Artix)
Search your drive with lsblk
```
lsblk
```
Use cfdisk to partition the drive
```
cfdisk /dev/sdx #where sdx must be changed with corresponding drive name
```
Create an Efi,swap and Root partition
```
New -> 512M -> Type=EFI 
New -> 8G -> Type=Linux Swap
New -> *press enter to use the remaining space* 
Write -> type *yes*
Exit
```
Format the partitions
```
mkfs.fat -F 32 -n "EFI" /dev/sdx1 #Boot partition
mkswap /dev/sdx2 #Swap 
mkfs.btrfs -L ROOT /dev/sdx3
```
Create subvolumes
```
mount /dev/sdx3 /mnt
btrfs su cr /mnt/@
btrfs su cr /mnt/@home
btrfs su cr /mnt/@snapshots
umount /mnt
```
Mount the root partition and subvolumes (root,home,.snapshots)
```
mount /dev/sdx3 /mnt

mount -o noatime,nodiratime,compress=zstd,commit=120,space_cache=v2,ssd,discard=async,autodefrag,subvol=@ /dev/sdx3 /mnt
mount --mkdir -o noatime,nodiratime,compress=zstd,commit=120,space_cache=v2,ssd,discard=async,autodefrag,subvol=@home /dev/sdx3 /mnt/home
mount --mkdir -o noatime,nodiratime,compress=zstd,commit=120,space_cache=v2,ssd,discard=async,autodefrag,subvol=@snapshots /dev/sdx3 /mnt/.snapshots
```
Mount the EFI partition and enable swap
```
mount --mkdir /dev/sdx1 /mnt/boot/efi
swapon /dev/sdx2
```
Install base system
### Arch
```
pacstrap -K /mnt base base-devel linux linux-firmware linux-headers networkmanager sudo vim man-db man-pages texinfo git btrfs-progs zstd
```
### Artix
```
basestrap /mnt base base-devel linux linux-firmware linux-headers linux-lts linux-lts-headers dinit elogind-dinit networkmanager git networkmanager networkmanager-dinit btrfs-progs zstd man vim 
```
If you need drivers for ur machine and CPU microcode (in my case it's AMD)
```
pacstrap -K /mnt amd-ucode amdgpu mesa vulkan-radeon libva-mesa-driver mesa-vdpau xf86-video-amdgpu
```
```
basestrap -K /mnt amd-ucode amdgpu mesa vulkan-radeon libva-mesa-driver mesa-vdpau xf86-video-amdgpu
```
Generate fstab
### Arch
```
genfstab -U /mnt >> /mnt/etc/fstab
```
### Artix
```
fstabgen -U /mnt >> /mnt/etc/fstab
```
We can finally chroot into our new system!
```
arch-chroot /mnt #For arch installation
artix-chroot /mnt #For artix installation
```
Set locale
### Edit locale.gen and decomment your locale (in my case it's it_IT.UTF-8)
```
vim /etc/locale.gen
#after decommenting run:
locale-gen
```
Set locale permanently and console keyboard mapping
```
echo "LANG=it_IT.UTF-8" >> /etc/locale.conf
echo "KEYMAP=it" >> /etc/vconsole.conf
```
Set root password
```
passwd
```
Set timezone
```
ln -sf /usr/share/zoneinfo/Europe/Rome /etc/localtime
hwclock --systohc
```
Replace hostname with your favorite name
```
echo "archPeak" >> /etc/hostname #or artixPeak
```
Modify hosts and write the following
```
vim /etc/hosts

127.0.0.1    localhost
::1          localhost
127.0.1.1    archPeak.localdomain  archPeak #or artixPeak
```
Create new user and add it to groups (specifically wheel)
```
useradd -mG audio,video,input,network,wheel,storage laywork #replace laywork with your favorite name
passwd laywork
```
#modify sudoers
```
EDITOR=vim visudo #replace vim with your favorite editor
```
and decomment #wheel ...
to enable privileges as a superuser for your user
### ONLY FOR ARTIX
Add arch repositories\
```
#Install these programs
pacman -S archlinux-keyring artix-archlinux-support
#and run these command
pacman-key --populate archlinux
```
Modify /etc/pacman.conf and add these lines
```
[lib32]
Include = /etc/pacman.d/mirrorlist

# stable arch repos
[extra]
Include = /etc/pacman.d/mirrorlist-arch
[multilib]
Include = /etc/pacman.d/mirrorlist-arch
```
Also decomment something where color is mentioned (I don't remember atm), everything will look more beatiful :)

Finally let's install other packages
```
pacman -Syy grub os-prober grub-btrfs inotify-tools timeshift pipewire pipewire-alsa pipewire-jack pipewire-pulse wireplumber pipewire-dinit pipewire-pulse-dinit wireplumber-dinit
```
Let's install grub (I'll switch to refind when I'll understand how to fucking make that shit work)
```
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```
### Enable NetworkManager
For artix it's better to enable after reboot because during chroot, the commands of dinit are a bit different:
```
sudo dinitctl enable NetworkManager #after reboot
```
For arch it's basically the same but with systemd (ew)
```
sudo systemctl enable NetworkManager
```
Exit from chroot, unmount everything and reboot
```
exit
umount -R /mnt
reboot
```
Enable and start the syncronization service on arch
```
sudo timedatectl set-ntp true
```
