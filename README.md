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
Create subvolume

Mount the partitions
mount -o noatime,compress=zstd



