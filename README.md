# Arch Installation

Instructions for myself.

### Update keyring (Sometimes (especially if the ISO is older), the keyring is outdated and installation might not work. Very rare tho, but never hurts to update it.)

```sh
pacman -Sy archlinux-keyring
```

### If using archinstall, then updating archinstall can sometimes cause [issues](https://github.com/archlinux/archinstall/issues/2490#issuecomment-2561474713), [(2)](https://www.reddit.com/r/archlinux/comments/w1pmlz/comment/jer48b4/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button) so only update it if it doesn't work properly without updating, or if the latest version has a feature you need. If you do update, and run into issues, you might need to update more packages. See the issues linked above for help

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
sgdisk -n1:0:+1G -t1:ef00 -c1:EFI -N2 -t2:8304 -c2:ROOT /dev/nvme0n1
```

- n(n) = partition number n with start and end sector (1gb in my case)
- N(n) = partition number n with largest available size
- t(n) = type for partition number n (ef00 for EFI and 8304 for linux x86-64 in my case)
- c(n) = name of partition number n. (Not necessary to name a partition.)

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
mkfs.fat -F 32 /dev/nvme0n1p1
```

Swap

```sh
mkswap /dev/nvme0n1p2
swapon /dev/nvme0n1p2
```

Root

```sh
mkfs.ext4 /dev/sda3
```

### Mounting

Boot (EFI)

```sh
mount --mkdir /dev/nvme0n1p1 /mnt/boot
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

### Base installation

```sh
pacstrap -K /mnt base linux linux-firmware sof-firmware base-devel intel-ucode e2fsprogs dosfstools nano neovim less man-db man-pages fastfetch networkmanager openssh os-prober refind git bash-completion mtools efibootmgr gptfdisk dkms reflector expac pacman-contrib
```

- Replace `linux` with the [kernel](https://wiki.archlinux.org/title/Kernel) of your choice.
- You can exclude the `linux-firmware` and `sof-firmware` packages if installing in a VM. `sof-firmare` is usually only needed for audio support in laptops.
- Replace `intel-ucode` with `amd-ucode` if you have an AMD CPU.
- If you chose file systems other than "ext4" and "VFAT", then replace `e2fsprogs` and `dosfstools` with the packages needed for your chosen file system. See [File systems](https://wiki.archlinux.org/title/File_systems#Types_of_file_systems)
- Exclude `os-prober` if not dual booting.
- Replace `refind` with the bootloader of your choice.
- Replace `nano` and `neovim` with [text editor(s)](https://wiki.archlinux.org/title/List_of_applications/Documents#Text_editors) of your choice

### . Generate filesystem table

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
127.0.0.1 localhost
::1  localhost
127.0.0.1 arch.localdomain arch
```

### Enable parallel downloads

Enabled by default now.

```sh
nano /etc/pacman.conf
```

Add eye candy in pacman:

```sh
nano /etc/pacman.conf
```

Under the `Misc options` section, uncomment `ParallelDownloads`, `VerbosePkgLists` and `Color`
and add `ILoveCandy`

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

### Install bootloader (I'm using refind, these installation instructions are modified because when installing refind from chroot, [it adds kernel parameters of the iso instead of our installed system.](https://wiki.archlinux.org/title/REFInd#Installation_with_refind-install_script) )

```sh
refind-install --usedefault /dev/nvme0n1p1 --alldrivers
mkrlconf
nano /boot/refind_linux.conf
```

remove first two lines

```sh
nano /boot/EFI/BOOT/refind.conf
```

go to archlinux section and replace UUID with path to efi partition (e.g. /dev/nvme0n1p1)

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

uncomment `%wheel ALL=(ALL) ALL`
or uncomment this for no password: `%wheel ALL=(ALL) NOPASSWD: ALL`
add this line to disable repeated password prompts: `Defaults !tty_tickets`

### Enable SSD trimer (ignore if you have HDD)

```sh
systemctl enable fstrim.timer
```

### Enable networkmanager

```sh
systemctl enable NetworkManager
```

# end

---

# Credits

- [The Arch Wiki](https://wiki.archlinux.org/title/Installation_guide)
- [Arch install guide by D3SOX](https://arch.d3sox.me/) (By far the best arch install guide for beginners I've ever come across.)
- A whoooole lot of youtube videos
- My own experience of reinstalling arch a bunch of times since I first started using Linux (November 2024).

---

# To-do

- instructions for VirtualBox and VMWare
- fixing resolution in VMs using bootloader kernel parameters
- instructions for zram generator instead of swap partition
- btrfs snapper compatible layout
- sudo pacman -S dkms avahi eza git wget efivar
- pacman -S rebuild-detector
- yay -S downgrade
- writing automated script and maybe ask the user for somethings so other people can also use this script, give it a cool name like archfi, archery and other great scripts lol.
