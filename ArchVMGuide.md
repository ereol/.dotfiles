# All command lines to install Arch on VM,

Download and use Arch Linux ISO file from torrent through [here](https://archlinux.org/download/). 
...This is best done following the [Arch Installation Guide.](https://wiki.archlinux.org/title/Installation_guide)

To turn boot to UEFI, navigate to where the VM is located and add this line to the .vmx file
..`firmware = "efi"`

To verify if you are in UEFI, check if files are generated using
..`$ ls /sys/firmware/efi/efivars`

The rest of the document are fairly straightforward and should be understandable
..`$ ping -c 4 www.google.org`
..`$ timedatectl set-ntp true`

This checks the disk in preparation for partition, ISO will most likely be sr0 or loop0 and disk will be sda with 'x' amount of GB
..`$ lsblk`

We will use cfdisk, a GUI version of fdisk as its user-friendly, we will now prepare to create out partition in sda. This will consist of 'FAT32 EFI' boot partition, an 'ext4 root (/) partition and aswap partition.
..`$ cfdisk /dev/sda`

Select
1. Select `'gpt'` for label and press `enter`
2. Select `'new'` and type `'512M`' to create EFI partition
3. Select `'type'` and change partition to `'EFI Systems'`
4. Select `'new'` to create root partition (sda2) and `enter desired amount`, leaving space, minimum 1G, usually 4G for 'swap' partition
5. Arrow down and select `'new'` on free space to create the swap partition (sda3) then `enter remaining amount of storage`
6. Select `'type'` then select `'Linux swap'`
7. Arrow to `'write'` then write `'yes'` to confirm you want to write the partition
8. Select `'quit'` and enter to exit cfdisk

To verify the partitions have been created, do `lsblk`

Now we need to create approrpiate file systems starting with swap, root then EFI
..`$ mkswap /dev/sda3`
..`$ swapon /dev/sda3`

..` $mkfs.ext4 /dev/sda2`

..`$ mkfs.fat -F32 /dev/sda1`

Now we mount the file systems to proceed with installation
..`$ mount /dev/sda2 /mnt`
..`$ mkdir /mnt/boot`
..`$ mount /dev/sda1 /mnt/boot`

Now we install essentials making up Arch
..`$ pacstrap /mnt base linux linux-firmware`
 
After finishing the download (takes a few minutes), create ftsba file so the system knows where to mount the partitions for boot.
..`$ genfstab -U /mnt >> /mnt/etc/fstab`

Now we chroot it
..`$ arch-chroot /mnt`

Now we customise our localisation settings, An example is `ln -sf /usr/share/zoneinfo/Australia/Melbourne /etc/localtime`
..`$ ln -sf /usr/share/zoneinfo/Region/City /etc/localtime`

Now we install vim via pacman
..`$ pacman -S vim`

now you vim `/etc/locale.gen` and uncomment any locale needed to use. An example is en_AU.UTF-8 UTF-8. Then generate locale using this command
..`$ locale-gen`

Now create `locale.conf` and set up language by adding such as en_AU.UTF-8 UTF-8
..`$ vim /etc/locale.conf`

Now vim into `/etc/hostname` and add desired hostname, hostname is computer name for clarification.

Next, edit `/etc/hosts` file with chosen hostname. Entries should be similar to this
..`127.0.0.1   localhost
::1         localhost
127.0.1.1   archvm.localdomain  archvm`

Next we configure networking for our Arch VM
..`$ systemctl enable systemd-networkd`
..`$ systemctl enable systemd-resolved`

Determine your network interface through
..`$ ip addr`

Use ens33 for name in next step, vim /etc/systemd/network/20-wired.network/ and add the following
..`[Match]
Name=ens33

[Network]
DHCP=yes`

Set password for 'root' user
..`$ passwd`

If using intel, install Intel-microcode
..`$ pacman -S intel-ucode`

The final step is installing bootloader. Grub will be used here
..`$ pacman -S grub efibootmgr`

Installing bootloader to EFI partition with this command
..`$ grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB`

Generate grub configuration file
..`$ grub-mkconfig -o /boot/grub/grub.cfg`

Base installation is finished! Now you can unmount and reboot the system.
..`$ # exit
# umount -R /mnt
# reboot`




