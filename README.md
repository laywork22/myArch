# ArchLinux/ArtixLinux(dinit) basic installation
## Basic installation

###Login as root (Artix) 
> sudo su

### Partioning (Arch/Artix)
Search your drive with lsblk
> lsblk 

Use cfdisk to partition the drive
> cfdisk /dev/sdx #where sdx must be changed with corresponding drive name
Create an Efi and Root partition
> New -> 512M -> Type=EFI \n
> New -> *press enter to use the remaining space* \n
> Write -> type *yes* \n
> Exit \n

