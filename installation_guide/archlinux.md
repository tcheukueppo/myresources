# arch linux installation guide

## PART ONE: BASIC INSTALLATION

1. Download archlinux iso from archlinux.org

2. Verify signature

On any linux system
```bash
gpg --keyserver-options auto-key-retrieve --verify archlinux-version-x86_64.iso.sig
```
or on an already installed os,  arch you can type
```bash
pacman-key -v archlinux-version-x86_64.iso.sig
```

3. Burn the downloaded iso into a optical disc or use a USB stick, for the usb
stick(in my case), this can be done with `ventoy` or through the `dd` command.

With `dd` do
```bash
dd if=/path/to/the/iso of=/path/to/the/devices
```

with `ventoy` do
```bash
ventoy ...
```

After doing this, connect your usb stick in to an appropiate port of your pc,
boot it(note that the boot order can be changed through the **BIOS**)
Entering the boot mode of you pc can ussualy be triggered either through
one of the f{1..11} keys.

4. Set the keyboard layout

For of keyboard layouts can be found here
```bash
ls /usr/share/kbd/keymaps/**/*.map.gz
```

so to modify keyboard layout, do
```bash
loadkeys layout_name
```
where layout_name is the name which correspond to the file in
`/usr/share/kbd/keymaps/**/` with its extension omitted

5. Verify the boot mode

try this
```bash
ls /sys/firmware/efi/efivars
```
Now get this: if the command list the content of `efivars` directory 
with no errors, then the system is booted in *EFI* mode but if the
directory doesn't exist, then the system is instead boot in BIOS mode.

6. Connecting to the internet

use
```bash
ip link
```
to list the possible network interface, either use
an ethernet cable(plug it in), iwctl(for wifi), mmcli(for mobile broadband modem) to
connect to the internet

configure dhcp(systemd-networkd), dns(system-resolved).

verify connection via
```bash
ping archlinux.org
```

7. Update system clock through

```bash
timedatectl set-ntp true
```

8. Partition your disk via `fdisk` utility

(skip this if already did on another OS)

After partitioning, mount your choosen partition with

```bash
mount /dev/sdX /mnt
```

9. Create a swap patition with the following commands

```bash
mkswap /dev/sdY
swapon /dev/sdY
```

10. Install essential packages via `pacstrap` script

```bash
pacstrap /mnt base linux linux-firmware
```

## PART TWO: SYSTEM CONFIGURATION

11. Generate fstab

use -U(recommended) or -L flag to identify the partition
either by it UUID or LABEL respectively

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

12. Chroot into your system

```bash
arch-chroot /mnt
```

13. Timezone

Set up your timezone
```bash
ln -sf /usr/share/zoneinfo/YourRegion/YourCity /etc/localtime
```
Run hwclock to generate `/etc/adjtime`
```bash
hwclock --systohc
```

14. Locailization

Edit /etc/locale.gen and uncomment en_US.UTF-8 UTF-8 and other
needed locales. Generate the locales by running: 

```bash
locale-gen
```

Create the locale.conf(5) file, and set the LANG variable accordingly:

in `/etc/locale.conf`

```bash
LANG=en_US.UTF-8
```

If you set the keyboard layout, make the changes persistent in vconsole.conf(5)

in `/etc/vconsole.conf` put

```bash
KEYMAP=de-latin1
```

15. Network configuration

Create the hostname file:

in `/etc/hostname` put

`myhostname`

Add matching entries to hosts(5):

in `/etc/hosts`

```bash
127.0.0.1	localhost
::1		localhost
127.0.1.1	myhostname.localdomain	myhostname
```

If the system has a permanent IP address, it should be used instead of `127.0.1.1`.

Complete the network configuration for the newly installed environment,
that may include installing suitable network management software.

16. Initramfs

Creating a new initramfs is usually not required, because mkinitcpio
was run on installation of the kernel package with pacstrap.

For LVM, system encryption or RAID, modify mkinitcpio.conf(5) and
recreate the initramfs image:
```bash
mkinitcpio -P
```

17. Root password

```bash
passwd
```
