# ArchLinux/ArtixLinux(dinit) basic installation

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

mount -o noatime,nodiratime,compress=zstd,commit=120,space_cache=v2,ssd,discard=async,autodefrag,subvol=@ /mnt
mount --mkdir -o noatime,nodiratime,compress=zstd,commit=120,space_cache=v2,ssd,discard=async,autodefrag,subvol=@home /mnt/home
mount --mkdir -o noatime,nodiratime,compress=zstd,commit=120,space_cache=v2,ssd,discard=async,autodefrag,subvol=@snapshots /mnt/.snapshots
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
If you need drivers for ur machine
## AMD






