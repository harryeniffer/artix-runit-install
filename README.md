# Harry's epic BIOS encrypted Artix Linux install for the Runit init system

login with artix, artix
lsblk and verify your drive names
placeholders will be used such as DISK, HOSTNAME, YOUR_TEXTEDITOR etc. 

dd if=/dev/urandom of=/dev/DISK # completely wipe data on the disk

fdisk /dev/DISK #format the disk
	
	n
	p
	1
	DEFAULT
	+1G # boot partition
	n
	p
	2
	DEFAULT # use the remaining drive space for root and home, no /home partition
	DEFAULT
	w

mkfs.fat -F32 /dev/DISK1 # format the boot partition, this could be sda1, nvme0n1p1, etc.

cryptsetup luksFormat /dev/DISK2 # encrypt the root partition

	YES
	ENTER PASSPHRASE FOR DECRYPTION
	VERIFY PASSPHRASE FOR DECRYPTION

cryptsetup open /dev/DISK2 ENCRYPTEDNAME # encryptedname is just the name of the luks partition, this can be anything

mkfs.btrfs /dev/mapper/ENCRYPTEDNAME # format the luks partition

mount /dev/mapper/ENCRYPTEDNAME /mnt # tradition from here https://man.archlinux.org/

mkdir /mnt/boot

mount /dev/DISK1 /mnt/boot

basestrap -i /mnt base base-devel runit elogind-runit linux linux-firmware grub networkmanager networkmanager-runit cryptsetup lvm2 lvm2-runit YOUR_TEXTEDITOR

artix-chroot /mnt bash

ln -s /usr/share/zoneinfo/Europe/London /etc/localtime # timezone for me, use America/New_York etc. for you

hwclock --systohc 

YOUR_TEXTEDITOR /etc/locale.conf

input the following

export LANG="en_US.UTF-8"
export LC_COLLATE="C"

save the file

YOUR_TEXTEDITOR /etc/locale.gen

uncomment your choice of locale, for example uncomment:

en_US.UTF-8 UTF-8
en_US ISO-8859-1

save the file

locale-gen

echo "HOSTNAME" > /etc/hostname # HOSTNAME can be whatever you like, its just the name of the system, as you are using echo, include the speech marks

YOUR_TEXTEDITOR /etc/hosts

input the following

127.0.0.1	localhost
::1		localhost
127.0.1.1	HOSTNAME.localdomain HOSTNAME

save the file

ln -s /etc/runit/sv/NetworkManager /etc/runit/runsvdir/current

passwd

	ENTER ROOT PASSWORD
	VERIFY ROOT PASSWORD

useradd -G wheel -m USERNAME

passwd USERNAME # this user will not have sudo permissions by default

	ENTER USER PASSWORD
	VERIFY USER PASSWORD
	
YOUR_TEXTEDITOR /etc/runit/sv/agetty-tty1/conf

on line 4, add the following:
	
GETTY_ARGS="--noclear --autologin USERNAME"
	
save the file 

YOUR_TEXTEDITOR /etc/mkinitcpio.conf

find the HOOKS line, it should look something like this: HOOKS=(base undev autodetect modconf) etc. 
after "block", add "encrypt" and "lvm2"
save the file

mkinitcpio -p linux

exit

lsblk -f >> /mnt/etc/default/grub

fstabgen -U /mnt >> /mnt/etc/fstabgen

artix-chroot /mnt bash

YOUR_TEXTEDITOR /etc/default/drub

go to the bottom of the file and find the information you outputted.
you only need the UUIDs of crypto_LUKS and btrfs
YOURCRYPTUUID is the UUID of the encrypted partition
YOURROOTUUID is the UUID of the root partition
find GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet"
add the following with your UUIDs.

GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet cryptdevice=UUID=YOURCRYPTUUID:cryptlvm root=UUID=YOURROOTUUID"

save the file

grub-install /dev/DISK # do not install onto a partition, install over the entire disk

grub-mkconfig -o /boot/grub/grub.cfg

exit

# this is the end of my epic artix encrypted install. for a basic dwm install, visit https://github.com/harryeniffer/basic-dwm-install

