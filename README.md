# archinstall
instructions for myself

### Update keyring (Sometimes (if the ISO is older) keyrings are outdated and installation might not work. Very rare tho)
```sh
pacman -Sy archlinux-keyring
```

### If using archinstall, then updating archinstall can sometimes cause [issues](https://github.com/archlinux/archinstall/issues/2490#issuecomment-2561474713), [(2)](https://www.reddit.com/r/archlinux/comments/w1pmlz/comment/jer48b4/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button) so only update it if it doesn't work properly without updating. If you update, and run into issues, you might need to update more packages. See the issues linked above for help.
```sh
pacman -Sy archinstall archlinux-keyring
```

### Verify boot mode
```sh
cat /sys/firmware/efi/fw_platform_size
```

### Time
```sh
timedatectl
timedatectl set-ntp true
```

### List partition table
```sh
lsblk
```

### Start partitioning

Fastest way is to use to sgdisk
```sh
sgdisk -n1:0:+1G -t1:ef00 -c1:EFI -N2 -t2:8304 -c2:ROOT /dev/sda
```
* n(n) = partition number n with start and end sector (1gb in my case)
* N(n) = partition number n with largest available size
* t(n) = type for partition number n (ef00 for EFI and 8304 for linux x86-64 in my case)
* c(n) = name of partition number n. (Not necessary to name a partition.)

cgdisk method

```sh
cgdisk
```
Code for EFI
```sh
ef00
```
Code for swap
```sh
8200
```
Code for Linux x86-64 root
```sh
8304
```

### Formatting
Boot (EFI)
```sh
mkfs.fat -F 32 /dev/sda1
```
Swap
```sh
mkswap /dev/sda2
swapon /dev/sda2
```
Root
```sh
mkfs.ext4 /dev/sda3
```

### Mounting
Boot (EFI)
```sh
mount --mkdir /dev/sda1 /mnt/boot
```
Root
```sh
root = mount /dev/sda3 /mnt
```

### Mirrors
```sh
reflector --country India --latest 5 --protocol http --protocol https --sort rate --save /etc/pacman.d/mirrorlist
pacman -Syy
```
> [!WARNING]  
> There's no need to include the "u" after the "syy" in the above command because we are in a live ISO and can't upgrade the system. But on an already installed system, you MUST include the "u" in every "Syu" or "Syyu" command. [Read more here on why you should never do partial upgrades.](https://wiki.archlinux.org/title/System_maintenance#Partial_upgrades_are_unsupported)

### Base installation (replace "refind" with bootloader of your choice)
```sh
pacstrap -K /mnt base linux linux-firmware sof-firmware base-devel intel-ucode e2fsprogs dosfstools nano neovim less man-db man-pages fastfetch networkmanager openssh os-prober refind git bash-completion mtools efibootmgr gptfdisk dkms reflector expac pacman-contrib
```

### Generate filesystem table
```sh
genfstab -U /mnt >> /mnt/etc/fstab
```

### Change root into the installation
```sh
arch-chroot /mnt
```

### Set root password
```sh
passwd
```

### Set time
```sh
ln -sf /usr/share/zoneinfo/Asia/Karachi /etc/localtime
hwclock --systohc
```

### Set local
```sh
nano /etc/locale.gen
```
uncomment the locale
```sh
locale-gen
echo LANG=en_US.UTF-8 > /etc/locale.conf
export LANG=en_US.UTF-8
```

### Setup Host (Computer) name (Replace "arch" with your desired hostname)
```sh
echo arch > /etc/hostname
nano /etc/hosts
```
add these lines:
```sh
127.0.0.1	localhost
::1		localhost  
127.0.0.1	arch.localdomain	arch
```

### Enable parallel downloads
```sh
nano /etc/pacman.conf
```

Add eye candy in pacman:
```sh
nano /etc/pacman.conf
```
Under the ```Misc options``` section, uncomment ```ParallelDownloads```, ```VerbosePkgLists``` and ```Color```
and add ```ILoveCandy```

### Setup multilib repos (32 bit libs needed for games and stuff)
```sh
nano /etc/pacman.conf
```
uncomment these two lines:
```sh
[multilib]
Include = /etc/pacman.d/mirrorlist
```
Then refresh the repos
```sh
pacman -Syu
```

### Install bootloader (I'm using refind, these installation instructions are modified because when installing refind from chroot, [it adds kernel parameters of the iso instead of our instlled system.](https://wiki.archlinux.org/title/REFInd#Installation_with_refind-install_script)  
```sh
refind-install --usedefault /dev/sda1 --alldrivers
mkrlconf
nano /boot/refind_linux.conf
```
remove first two lines
```sh
nano /boot/EFI/BOOT/refind.conf
```
go to archlinux section and replace UUID with path to efi partitio (e.g. /dev/sda1)

### Add users (replace "user" with your username)
```sh
useradd -m -G wheel,sys,log,rfkill,lp,adm -s /bin/bash user
```
Set password for user (replace "user" with your username)
```sh
passwd user
```

### Give sudo privileges to user
```sh
EDITOR=nano visudo
```
uncomment ```%wheel ALL=(ALL) ALL```
or uncomment this for no password: ```%wheel ALL=(ALL) NOPASSWD: ALL```
add this line to disable repeated password prompts: ```Defaults !tty_tickets```

### Enable SSD trimer (ignore if you have HDD)
```sh
systemctl enable fstrim.timer
```

### Enable networkmanager
```sh
systemctl enable NetworkManager
```

# end

---------------------------------------------------------------------------------------------------------------------------------
# Credits
* [The Arch Wiki](https://wiki.archlinux.org/title/Installation_guide)
* [Arch install guide by D3SOX](https://arch.d3sox.me/) (By far the best arch install guide for beginners I've ever came across)
* A whoooole lot of youtube videos
* My own experience of reinstalling arch a bunch of times since I first started using Linux (November 2024).


---------------------------------------------------------------------------------------------------------------------------------
# Work in progress, ignore.
* instructions for VirtualBox and VMWare
* fixing resolution in VMs using bootloader kernel parameters
* instructions for zram generator instead of swap partition
* btrfs snapper comaptible layout
* sudo pacman -S dkms avahi eza git wget
* pacman -S rebuild-detector
* yay -S downgrade
* writing automated script and maybe ask the user for somethings so other people can also use this script, give it a cool name like archfi, archery and    other great scripts lol.
