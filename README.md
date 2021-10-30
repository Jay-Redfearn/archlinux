# Arch Linux Install

Installation guide for Arch Linux.

# Preparation

I first downloaded the Arch Linux iso by going to Arch Linux Wiki and subsequently going to a mirror page to download the file from an HTTP source.

I added the iso to VMWare and chose the operating system for the virtual machine as “Other Linux 5.x and later kernel 64-bit”. When specified for the boot firmware, I selected “UEFI”.

It was not necessary for me to add firmware=”efi” to my Arch Linux.vmx file as VMWare automatically added the code when I selected the boot firmware as “UEFI”.

I booted into the VM after it was set up, and clicked on Arch Linux install medium (x86_64, UEFI). After booting in, verify the boot mode is UEFI by typing “ls /sys/firmware/efi/efivars”.

Network services should already be installed as VMWare bridges the current host’s network to the VM through Network Address Translation (NAT).

I corrected the time’s clock to the current time by typing “timedatectl set-ntp true”. I then checked the status of the system clock by typing “timedatectl status”. The clock was not showing in local time, so I viewed the available timezones with the “timedatectl list-timezones” command, and then changed the timezone to the central time zone by typing “timedatectl set-timezone America/Chicago”.


# Partitioning

Start the creating of partitions by typing “cfdisk /dev/sda”. Select “gpt” as the label type.

Select free space, hit the enter key when “New” at the bottom of the screen is selected, and choose the first partition size at 500M. Change the first partition size’s type to EFI System by entering “Type” at the bottom of the screen and clicking on the “EFI System” option.

Select the next device that says “Free space”, hit the enter key when “New” is selected, and choose the second partition size at 4G. Enter “Type” and click on the “Linux swap” option.

Select the next device that says “Free space”, hit the enter key when “New” is selected, and choose the third partition size at whatever space is left over (in my case, 20.5G). Enter “Type” and click on the “Linux filesystem” option.

Select “Write” at the bottom of the screen and hit the enter key. When promoted “Are you sure…”, type “yes”.

Verify the three partitions are created by typing “lsblk” in the terminal. sda1, sda2, and sda3 should be visible.

Set up the first partition (sda1, the boot partition) as FAT32 by typing “mkfs.fat -F32 /dev/sda1”.

Set up the second partition (sda2, the swap partition) as swap space by typing “mkswap /dev/sda2” and subsequently typing “swapon /dev/sda2”.

Set up the third partition (sda3, the root filesystem) as EXT4 by typing “mkfs.ext4 /dev/sda3”.

Mount the root partition by typing “mount /dev/sda3 /mnt”.

Type “mkdir /mnt/boot” and “mkdir /mnt/boot/efi” to create the directory where the boot partition will be stored. Mount the boot partition by typing “mount /dev/sda1 /mnt/boot/efi”.


# System Configuration

To create a mirror, first sync the pacman repository by typing “pacman -Syy”. Install reflector to get a list of fresh and fast mirrors in your country by typing “pacman -S reflector”. Make a backup of the current mirror list by typing “cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak”.

Replace the good mirror list over the current mirror list by typing “reflector -c "US" -f 12 -l 10 -n 12 --save /etc/pacman.d/mirrorlist”.

Use the pacstrap script to install necessary packages by typing “pacstrap /mnt base linux linux-firmware vim nano”.

Generate an fstab file to determine how disk partitions among other things are mounted into the filesystem by typing “genfstab -U /mnt >> /mnt/etc/fstab”.

Enter the mounted disk as root by typing “arch-chroot /mnt”. The console text will be changed to [root@archiso /].

Set the time zone again by typing “ln -sf /usr/share/zoneinfo/Region/City /etc/localtime” and subsequently “hwclock --systohc”.

Do cd ~.

Vim into locale.gen by typing “vim /etc/locale.gen” and uncomment “en_US.UTF-8 UTF-8” by removing the # to the left of the text. Generate locales that stores language and other formats for the system by typing “locale-gen”. 

Vim into locale.conf by typing “vim /etc/locale.conf” and add “LANG=en_US.UTF-8” to the file. This file contains our language.

Create a file called vconsole.conf by typing “echo "KEYMAP=us" > vconsole.conf”.


# Networking

Create a hostname by typing “echo jredfearn > /etc/hostname”. Create the hosts file by typing “touch /etc/hosts”. Edit the “/etc/hosts” file and add…
	“127.0.0.1 localhost
	::1 localhost
	127.0.1.1 jredfearn.localdomain jredfearn”

Install the network manager by typing “pacman -S networkmanager”. Enable the manager on boot by typing “systemctl enable NetworkManager”.


# Boot Loader

Change root password by typing “passwd” and entering a password when prompted.

Install Intel microcode if using an Intel processor “pacman -S intel-ucode”.

Install a bootloader by typing “pacman -S grub efibootmgr” and subsequently “grub-install --target=x86_64-efi --efi-directory=/boot/efi”. Generate a grub configuration file by typing “grub-mkconfig -o /boot/grub/grub.cfg”.


# Creating Users

Install sudo by typing “pacman --sync sudo”.

Type the following commands…
  
  "useradd -m -G wheel jredfearn"
	"useradd -m -G wheel sal"
	"useradd -m -G wheel codi"
  
	“passwd jredfearn” and set a password for account
	“passwd sal” and set password as “GraceHopper1906”
	“passwd codi” and set password as “GraceHopper1906”
	“passwd -e sal” to expire account on next login
	“passwd -e codi” to expire account on next login

Install vi by typing “pacman -S vi”. Load into visudo by typing “visudo”. Look for the line that says “# %wheel ALL=(ALL) ALL” and remove the “#”. This allows all members of group wheel to execute any command.


# Desktop Environment

Install KDE by typing “pacman -S xorg”, “pacman -S plasma”, “pacman -S plasma-wayland-session”, “pacman -S kde-applications”.

If installing the kde-applications dependency fails, follow this next line of directions: Type “pacman -Syu --needed kde-applications” to fetch another mirror from mirror list to install kde applications. This will subsequently install kde-applications, so go through the prompts to install the dependency.

Enable the Display Manager by typing “systemctl enable sddm.service”. Enable the Network Manager by typing “systemctl enable NetworkManager.service”.

Shutdown the system by typing “shutdown now”.

Startup the system by activating it through VMWare.

Problems with installation of desktop environment: could not install all dependencies at once due to kde-applications failing due to a bad mirror; had to revert to one at a time to see which dependency was having installation issues.

# Install zsh

Install zsh by typing “sudo pacman -S zsh” and proceeding through the installation.


# Install ssh

Install ssh by typing “sudo pacman -S openssh”. Type “sudo systemctl start sshd” to active SSH. Type “sudo systemctl enable sshd” to allow SSH to automatically startup on boot.

EXTRA: The time was incorrect on the machine, so I reset it by going to terminal and typing “timedatectl set-ntp true”. This may have been due to reverting to previous snapshots.


# Color Code Terminal

Make up a backup of .bashrc by typing “cp .bashrc .bashrc.backup”.

Use Konqueror in the KDE to go to https://averagelinuxuser.com/linux-terminal-color/#google_vignette and download the bash.bashrc, DIR_COLORS, and .bashrc files.

Extract all files to the “home” directory (~/). If the system is wanting to overwrite .bashrc, overwrite the file as you already have a backup of the original file.

Move bash.bashrc by typing “sudo mv bash.bashrc /etc/bash.bashrc”.

Move DIR_COLORS by typing “sudo mv DIR_COLORS /etc/”.

Restart the terminal and you should have color coding.


# Create Aliases

Create aliases by typing “alias newcommand=’originalcommand’’.

I created these aliases with the following commands: “alias ..=’cd ..’” and “alias pingtest=’ping google.com’”.


# Install Chrome

First install the base-devel and git dependencies by typing “sudo pacman -S --needed base-devel git”.

Clone the google-chrome AUR repository by typing…
	“git clone https://aur.archlinux.org/google-chrome.git
cd google-chrome
makepkg -sri”

Proceed with the installation.

Note: To update Chrome, use…
	“git pull
makepkg -si”


# Extra

Install nano by typing “pacman -S nano”.

Install tree by typing “sudo pacman -S tree”.


# Sources

https://wiki.archlinux.org/title/Installation_guide

https://linuxconfig.org/install-arch-linux-in-vmware-workstation

https://itsfoss.com/install-arch-linux/

https://itsfoss.com/install-kde-arch-linux/

https://linuxhint.com/install_kde_arch_linux/

https://medium.com/tech-notes-and-geek-stuff/install-zsh-on-arch-linux-manjaro-and-make-it-your-default-shell-b0098b756a7a

https://linuxhint.com/install_ssh_server_on_arch_linux/

https://averagelinuxuser.com/linux-terminal-color/#google_vignette

https://www.cyberciti.biz/tips/bash-aliases-mac-centos-linux-unix.html

https://www.geeksforgeeks.org/how-to-install-google-chrome-in-arch-based-linux-distributions/


![image](https://user-images.githubusercontent.com/41350159/139554181-9f95d87b-c59c-490c-89fa-08fd94fb1107.png)
