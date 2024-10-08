#+LAYOUT: post
#+TITLE: Install Fest: Arch Linux
#+DATE: 2017-01-06
#+AUTHOR: Yash Srivastav
#+LIQUID: enabled
#+WEBSITE: https://yashsriv.org
#+SUBSECTION: installfest17
#+HIDDEN: true

\*\* Pre-installation

Arch Linux should run on any i686 or x86_64 compatible machine with a
minimum of 256 MB RAM, or 512 MB for x86_64. A basic installation with
all packages from the base group should take less than 800 MB of disk
space. As the installation process needs to retrieve packages from a
remote repository, a working internet connection is required.

_Note_ : All commands will look like this:
#+BEGIN_EXAMPLE

# some_command

#+END_EXAMPLE
The =some_command= portion is supposed to be typed after the prompt which is usually:
#+BEGIN_EXAMPLE
root@archiso ~ #
#+END_EXAMPLE

\*\*\*\* Set the keyboard layout
See Archwiki installation page if you really need this, i.e., your keyboard
is not working as expected.

\*\*\*\* Verify the boot mode

     If UEFI mode is enabled on an UEFI motherboard, Archiso will boot Arch
     Linux accordingly via systemd-boot. To verify this, list the efivars
     directory:

     #+BEGIN_EXAMPLE
     # ls /sys/firmware/efi/efivars
     #+END_EXAMPLE

\*\*\*\* Connect to the Internet

     For wired connection with *DHCP* (for example - academic area),
     just connecting a LAN cord to your ethernet port should start the connection.
     You can verify connection on iitk network with the following command:
     #+BEGIN_EXAMPLE
     # ping 172.31.1.1
     #+END_EXAMPLE

     For networks requiring static ip (such as your rooms), first stop
     dhcpcd service with ~systemctl stop dhcpcd@<TAB>~ (Here =<TAB>= refers to pressing the TAB key on the keyboard)
     and then see: [[https://wiki.archlinux.org/index.php/Netctl#Wired][ArchWiki Network Configuration]]

     For wireless connections, use:
     #+BEGIN_EXAMPLE
     # wifi-menu
     #+END_EXAMPLE
     to connect to available networks. You can check connection using the previous command.

\*\*\*\* Update the system clock

      Use timedatectl(1) to ensure the system clock is accurate:
     #+BEGIN_EXAMPLE
     # timedatectl set-ntp true
     #+END_EXAMPLE
      To check the service status, use ~timedatectl status~.

\*\*\*\* Partition the disks

     You might have already partitioned your disks from windows if you have seen
     the parent guide of this post. So, you will have 3 partitions:
       * One partition for the root directory / - approx 50-100gb.
       * A swap partition of approximately your RAM size.

\*\*\*\* Format the partitions

     Once the partitions have been created, each must be formatted with an
     appropriate file system. For a list of partitions use:
     #+BEGIN_EXAMPLE
     # lsblk -o name,size,type,mountpoint,fstype

     NAME                                                       SIZE TYPE MOUNTPOINT FSTYPE
     sda                                                      931.5G disk
     ├─sda1                                                     260M part            vfat
     ├─sda2                                                      16M part
     ├─sda3                                                   416.2G part            ntfs
     ├─sda4                                                     808M part            ntfs
     ├─sda5                                                      12G part            swap
     ├─sda6                                                      10G part            ext4
     ├─sda7                                                      50G part            ext4
     ├─sda8                                                   308.5G part            ext4
     ├─sda9                                                    68.4G part            ext4
     ├─sda10                                                     50G part            ext4
     ├─sda11                                                    694M part            ntfs
     └─sda12                                                   13.6G part            ntfs

     #+END_EXAMPLE

     Now, in the example above, my main harddisk is =/dev/sda=, yours can be anything. From now
     on, I will refer to it as =/dev/sdx= (=x= = variable corresponding to your harddisk).

     Now find the partitions you created. They will probably be the ones with the highest index
     (it's still your job to verify that).

     To format the main root partition (let it be /dev/sdxr) (the 50-100 gb one), use:
     #+BEGIN_EXAMPLE
     # mkfs.ext4 /dev/sdxr
     #+END_EXAMPLE
     where =r= is the number of the root partition from the output of ~lsblk~

     Swap partition is required if you have RAM less than 8 GB, else you can leave it out if you want.
     To format the swap partition (the partition with the same size as your RAM), use:
     #+BEGIN_EXAMPLE
     # mkfs.ext4 /dev/sdxs
     #+END_EXAMPLE
     where =s= is the number of the swap partition from the output of ~lsblk~

     To format the home partition, use:
     #+BEGIN_EXAMPLE
     # mkfs.ext4 /dev/sdxh
     #+END_EXAMPLE
     where =h= is the number of the home partition from the output of ~lsblk~

\*\*\*\* Mount the file systems

     Mount the root partition to =/mnt=, for example:
     #+BEGIN_EXAMPLE
     # mount /dev/sdxh /mnt
     #+END_EXAMPLE

     Find out if your computer uses UEFI or not. The best way (I know) is to verify
     whether you have a =vfat= partition (as in my case =/dev/sda1=).
     If yes, then:
     #+BEGIN_EXAMPLE
     # mkdir /mnt/boot
     # mkdir /mnt/home
     # mount /dev/sdxh /mnt/home
     # mount /dev/sdxe /mnt/boot
     #+END_EXAMPLE
     (Here =/dev/sdxe= is the =vfat= partition)
     This new partition which we are mounting to boot is Windows EFI Partition which containes Windows Boot Manager.
     Also remember this info(whether you have a UEFI system or not) for one of the future steps.

     Mount the swap partition:
     #+BEGIN_EXAMPLE
     # mkswap /dev/sdxs
     #+END_EXAMPLE

\*\* Installation

\*\*\*\* Select the mirrors

     Packages to be installed must be downloaded from mirror servers, which
     are defined in =/etc/pacman.d/mirrorlist=. Since we are in iitk, we will
     use the iitk mirrors. For that, use =nano=:
     #+BEGIN_EXAMPLE
     # nano /etc/pacman.d/mirrorlist
     #+END_EXAMPLE

     And insert these line at the very top just below the initial comments in this order:
     #+BEGIN_EXAMPLE
     Server = http://mirror.cse.iitk.ac.in/archlinux/$repo/os/$arch
     Server = http://ftp.iitm.ac.in/archlinux/$repo/os/$arch
     #+END_EXAMPLE

\*\*\*\* Install the base packages

     Use the pacstrap script to install the base package group and other useful stuff:
     #+BEGIN_EXAMPLE
     # pacstrap /mnt base dialog iw wpa_supplicant sudo
     #+END_EXAMPLE

\*\* Configure the system

\*\*\*\* Fstab

     Generate an fstab file:
     #+BEGIN_EXAMPLE
     # genfstab -U /mnt >> /mnt/etc/fstab
     #+END_EXAMPLE

\*\*\*\* Chroot

     Change root into the new system:
     #+BEGIN_EXAMPLE
     # arch-chroot /mnt
     #+END_EXAMPLE

\*\*\*\* Time zone

     Set the time zone (probably =Asia/Kolkata=, since you live in India):
     #+BEGIN_EXAMPLE
     # ln -s /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
     #+END_EXAMPLE

      Run hwclock(8) to generate =/etc/adjtime=:
     #+BEGIN_EXAMPLE
     # hwclock --systohc --localtime
     #+END_EXAMPLE

\*\*\*\* Locale

     Open =/etc/locale.gen= using =nano=:
     #+BEGIN_EXAMPLE
     # nano /etc/locale.gen
     #+END_EXAMPLE
     Go to the line and remove the first =#=:
     #+BEGIN_EXAMPLE
     #en_US.UTF-8 UTF-8
     #+END_EXAMPLE
     Generate localisations with (execute):
     #+BEGIN_EXAMPLE
     # locale-gen
     #+END_EXAMPLE

     Open =/etc/locale.conf= using =nano= and add the following line:
     #+BEGIN_EXAMPLE
     LANG=en_US.UTF-8
     #+END_EXAMPLE

\*\*\*\* Hostname

     Create the =/etc/hostname= file. A hostname is a name for your pc (You can set that to anything consisting of only letters):
     #+BEGIN_EXAMPLE
     myhostname
     #+END_EXAMPLE

     You will need to add a matching entry to =/etc/hosts= (the last line):
     #+BEGIN_EXAMPLE
     127.0.0.1       localhost.localdomain   localhost
     ::1             localhost.localdomain   localhost
     127.0.1.1       myhostname.localdomain  myhostname
     #+END_EXAMPLE

\*\*\*\* Root password

     Set the root password:
     #+BEGIN_EXAMPLE
     # passwd
     #+END_EXAMPLE

\*\*\*\* Boot loader

     If you have an Intel CPU, install the intel-ucode package
     #+BEGIN_EXAMPLE
     # pacman -S intel-ucode
     #+END_EXAMPLE

     Now, you need to remember if you have a UEFI system or not.

**\*** No UEFI

      #+BEGIN_EXAMPLE
      # pacman -S grub os-prober ntfs-3g
      # grub-install --target=i386-pc /dev/sdx
      # grub-mkconfig -o /boot/grub/grub.cfg
      #+END_EXAMPLE
      Please replace =x= with the character of your harddisk.

**\*** UEFI

      #+BEGIN_EXAMPLE
      # pacman -S grub os-prober efibootmgr ntfs-3g
      # grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub
      # grub-mkconfig -o /boot/grub/grub.cfg
      #+END_EXAMPLE

**\*** After
The above steps may sometimes fail to recognize windows. Don't panic, see the reboot section.

\*\*\*\* New User

     Now, its time to create a new user:
     #+BEGIN_EXAMPLE
     # useradd -m -G wheel -s /bin/bash <username>
     #+END_EXAMPLE
     Here, a new user was added with the username you give and default shell =/bin/bash=.
     Just changing the username should suffice for most people.
     To change the user's password:
     #+BEGIN_EXAMPLE
     # passwd the_username_you_just_set
     #+END_EXAMPLE

     Now setup =sudo= by typing ~visudo~. This opens up the sudo configuration file in =vim=.
     Press =<Shift> + g= to goto the end of the file. Now go up, till you see this line:
     #+BEGIN_EXAMPLE
     ## Uncomment the below line to allow members of group wheel to execute any command
     # %wheel ALL=(ALL) ALL
     #+END_EXAMPLE
     Now, carefully place your cursor on the =#= just before =%wheel= and press =x=.
     This will remove the =#=. It will now look like this:
     #+BEGIN_EXAMPLE
     ## Uncomment to allow members of group wheel to execute any command
      %wheel ALL=(ALL) ALL
     #+END_EXAMPLE
     Now type =:wq= to save and exit.

     This should give your user sudo rights.

\*\* Reboot

Exit the chroot environment by typing exit or pressing =Ctrl+D=.

Optionally manually unmount all the partitions with ~umount -R /mnt~:

Finally, restart the machine by typing ~reboot~. Now while booting choose
grub as the default boot option.

After booting, you will encounter a black screen with option to login.
You can now log in with your user.

\*\*\*\* Post Reboot GRUB Fix
If your Windows did not show up during boot, run this command and check if windows
shows up on a reboot:
#+BEGIN_EXAMPLE # grub-mkconfig -o /boot/grub/grub.cfg
#+END_EXAMPLE

\*\* Post-installation

See [[https://wiki.archlinux.org/index.php/General_recommendations][General Recommendations]] for system management directions and
post-installation tutorials (like setting up a graphical user
interface, sound or a touchpad).

For a list of applications that may be of interest, see [[https://wiki.archlinux.org/index.php/List_of_applications][List of applications]].
