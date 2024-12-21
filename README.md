# archinstall
instructions for myself

### Update keyring (Sometimes (if the ISO is older) keyrings are outdated and installation might not work. Very rare tho)
```sh
pacman -Sy archlinux-keyring
```

### If using archinstall, then it's good to update that too
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
### Base installation
pacstrap -K /mnt base linux linux-firmware sof-firmware base-devel intel-ucode e2fsprogs dosfstools nano neovim less man-db man-pages fastfetch networkmanager openssh os-prober refind git bash-completion mtools efibootmgr gptfdisk dkms reflector expac

### Generate filesystem table
genfstab -U /mnt >> /mnt/etc/fstab

### Change into the installation
arch-chroot /mnt

### Set root password
passwd

### Set time
ln -sf /usr/share/zoneinfo/Asia/Karachi /etc/localtime
hwclock --systohc

### Set local
nano /etc/locale.gen
uncomment the locale
locale-gen
echo LANG=en_US.UTF-8 > /etc/locale.conf
export LANG=en_US.UTF-8

### Setup hostname (computer's name)
echo arch > /etc/hostname
nano /etc/hosts
add these lines:

127.0.0.1   localhost
::1         localhost
127.0.1.1   arch.localdomain arch

### Setup multilib (32 bit libs needed for games and stuff)
nano /etc/pacman.conf
uncomment these two lines:
[multilib]
Include = /etc/pacman.d/mirrorlist

### Enable parallel downloads and eye candy in pacman
nano /etc/pacman.conf
Under "Misc" section, uncomment "ParallelDownloads", "VerbosePkgLists" and "Color"
and add: ILoveCandy
pacman -Syu

### Install bootloader
refind-install --usedefault /dev/sda1 --alldrivers
mkrlconf
nano /boot/refind_linux.conf
remove first two lines
nano /boot/EFI/BOOT/refind.conf
go to archlinux section and replace UUID with path to efi


### Add users
useradd -m -G wheel,sys,log,rfkill,lp,adm -s /bin/bash blackshine
passwd blackshine

### Give sudo privileges
EDITOR=nano visudo
uncomment %wheel ALL=(ALL) ALL
or uncomment this if for no password: %wheel ALL=(ALL) NOPASSWD: ALL
add this to disable repeated password prompts: Defaults !tty_tickets

### Enable SSD trimer
systemctl enable fstrim.timer

### Enable networkmanager
systemctl enable NetworkManager







 sudo pacman -S dkms avahi eza git wget
 pacman -S rebuild-detector
