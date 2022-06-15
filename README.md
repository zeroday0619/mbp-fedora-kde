# mbp-fedora-kde

[![Build Status](https://github.com/zeroday0619/mbp-fedora-kde/actions/workflows/build-iso.yml/badge.svg)](https://github.com/zeroday0619/mbp-fedora-kde/actions/workflows/build-iso.yml)

Fedora ISO with Apple T2 patches built-in (Macbooks produced >= 2018).

All available Apple T2 drivers are integrated with this iso. Most things work, besides those mentioned in [not working section](#not-working).

Kernel - <https://github.com/mikeeq/mbp-fedora-kernel>

Drivers:

- Apple T2 (audio, keyboard, touchpad) - <https://github.com/MCMrARM/mbp2018-bridge-drv>
- Apple SMC - <https://github.com/MCMrARM/mbp2018-etc>
- Touchbar - <https://github.com/roadrunner2/macbook12-spi-driver/tree/mbp15>

> Tested on: Macbook Pro 15,2 13" 2019 i5 TouchBar Z0WQ000AR MV972ZE/A/R1 && Macbook Pro 16,2 13" 2020 i5

```
Boot ROM Version:	220.270.99.0.0 (iBridge: 16.16.6571.0.0,0)
macOS Mojave: 10.14.6 (18G103)
```

## How to install

- Turn off secure boot and allow booting from external media - <https://support.apple.com/en-us/HT208330>
- Download .iso from releases section - <https://github.com/mikeeq/mbp-fedora/releases/latest>
  - If it's split into multiple zip parts, i.e.: `livecd.zip` and `livecd.z01` you need to download all zip parts and then
    - join split files into one and then extract it via `unzip`
      - <https://unix.stackexchange.com/questions/40480/how-to-unzip-a-multipart-spanned-zip-on-linux>
    - or extract downloaded zip parts directly using:
      - on Windows `winrar` or other supported tool like `7zip`
      - on Linux you can use `p7zip`, `dnf install p7zip` and then to extract `7za x livecd.zip`
      - on MacOS you can use
        - `the unarchiver` from AppStore: <https://apps.apple.com/us/app/the-unarchiver/id425424353?mt=12>
        - or you can install `p7zip` via `brew` `brew install p7zip` and use `7za x livecd.zip` command mentioned above
          - to install `brew` follow this tutorial: <https://brew.sh/>
- Next you can check the SHA256 checksum of extracted .ISO to verify if your extraction process went well
  - MacOS: `shasum -a 256 livecd-fedora-mbp.iso`
  - Linux `sha256sum livecd-fedora-mbp.iso`
  - please compare it with a value in `sha256` file available in github releases
- Burn the image on USB stick >=8GB via:
  - Fedora Media Writer
    - Download: <https://getfedora.org/en/workstation/download/>
      - MacOS <https://getfedora.org/fmw/FedoraMediaWriter-osx-latest.dmg>
        - if you're prompted that application is insecure/from unknown developer, check <https://support.apple.com/en-us/HT202491>
      - Windows <https://getfedora.org/fmw/FedoraMediaWriter-win32-latest.exe>
    - Please use Custom Image option when burning the downloaded and extracted ISO to USB drive
  - `dd`
    - Linux `sudo dd bs=4M if=/home/user/Downloads/livecd-fedora-mbp-201908181858.iso of=/dev/sdc conv=fdatasync status=progress`
    - MacOS
      - try to find under which `/dev/` your USB stick is available `sudo diskutil list`
      - check if any partitions from it are mounted `df -h`, if they are please unmount `sudo diskutil unmount /dev/disk2s1`
      - exec `sudo dd if=/Users/user/Downloads/livecd-fedora-mbp-201908181858.iso of=/dev/disk2 bs=4m`
      - if `dd` is not working for you for some reason you can try to install `gdd` via `brew` and use GNU dd command instead `sudo gdd bs=4M if=/Users/user/Downloads/livecd-fedora-mbp-201908181858.iso of=/dev/disk2 conv=fdatasync status=progress`

        ```bash
        # To install gdd via brew, execute
        brew install coreutils
        ```

      - don't worry if `dd` command execution is slow on MacOS, it can take a while due to XNU's poor I/O performance
  - `Rufus` (GPT)- <https://rufus.ie/>, if prompted use DD mode
  - Please don't use `livecd-iso-to-disk` as it's overwriting ISO default grub settings and Fedora will not boot correctly!
- Install Fedora
  - First of all I recommend to shrink (resize) macOS APFS partition and not removing macOS installation entirely from your MacBook, because it's the only way to keep your device up-to-date. macOS OS updates also contains security patches to EFI/Apple T2
    - HowTo: <https://www.anyrecover.com/hard-drive-recovery-data/resize-partition-mac/> # Steps to Resize Mac Partition
  - Boot Fedora Installer from USB drive directly from macOS boot manager. (You can boot into it by pressing and holding Option key (ALT key) after clicking the power-on button when your computer was turned off or on restart/reboot when Apple logo is shown on the screen).
    - There will be two/three boot options available, usually the last one works for me. (There are multiple boot options, because there are three different partitions in the ISO to make the ISO bootable on different set of computers: 1) ISO9660: with installer data, 2) fat32, 3) hfs+)
  - I recommend using standard partition layout during partitioning your Disk in Anaconda (Fedora Installer) as I haven't tested other scenarios yet. <https://github.com/mikeeq/mbp-fedora/issues/2

    - please create a separate partition for Linux EFI (Linux HFS+ ESP) as Anaconda installer requires separate partition on Mac devices and it'll be reformated to EFI (FAT32) during post-install scripts Anaconda's step (at the end of installation process).

    ```bash
      /boot/efi - 1024MB Linux HFS+ ESP
      /boot - 1024MB EXT4
      / - xxxGB EXT4
    ```

    ![anaconda partitioning](screenshots/anaconda-3.png)

  - There will be an error on `Installing bootloader...` step, click Yes - It's related to `efi=noruntime` kernel arg

    ![bootloader issue](screenshots/bootloader.png)

    ```bash
    # /tmp/anaconda.log
    13:39:49,173 INF bootloader.grub2: bootloader.py: used boot args: resume=UUID=8a64abbd-b1a3-4d4a-85c3-b73800e46a1e rd.lvm.lv=fedora_localhost-live/root rd.lvm.lv=fedora_localhost-live/swap rhgb quiet
    13:39:54,649 ERR bootloader.installation: bootloader.write failed: Failed to set new efi boot target. This is most likely a kernel or firmware bug.
    ```

- To install additional languages (only English is available out of the box), install appropriate langpack via dnf `dnf search langpacks`, i.e.: to install Polish language pack execute: `dnf install langpacks-pl`
- After login you can update kernel by running `sudo update_kernel_mbp`, more info here: <https://github.com/mikeeq/mbp-fedora-kernel#how-to-update-kernel-mbp>
- You can change mappings of ctrl, option keys (PC keyboard mappings) by creating `/etc/modprobe.d/hid_apple.conf` file and recreating grub config. All available modifications could be found here: <https://github.com/free5lot/hid-apple-patched>

  ```bash
  # /etc/modprobe.d/hid_apple.conf
  options hid_apple swap_fn_leftctrl=1
  options hid_apple swap_opt_cmd=1

  sudo -i
  # Refresh grub and dracut config by executing update_kernel_mbp script
  update_kernel_mbp
  ```

- To change function key mappings for models with touchbar see `modinfo apple_ib_tb` and use `echo 2 > /sys/class/input/*/device/fnmode` instead of the `hid_apple` options. See
    [this issue](https://github.com/mikeeq/mbp-fedora-kernel/issues/12)
- Setup wifi and other model specific devices by following guides on `wiki.t2linux.org` - <https://wiki.t2linux.org/guides/wifi/>

## How to upgrade current mbp-fedora installations

```bash
# Docs: https://docs.fedoraproject.org/en-US/quick-docs/dnf-system-upgrade/
sudo -i

# Upgrade kernel beforehand
## update_kernel_mbp has built-in selfupgrade function, so when it fails it's just due to script update - please rerun everything should be good on second run
KERNEL_VERSION="5.17.6-f36" UPDATE_SCRIPT_BRANCH="v5.17-f36" update_kernel_mbp

# Upgrade your OS
dnf upgrade -y --refresh
dnf install -y dnf-plugin-system-upgrade

# Exclude official kernel from upgrade to not override mbp-fedora-kernel
## If you're trying to upgrade older version of mbp-fedora to latest version, please repeat a process by upgrading only to one major release of Fedora, i.e.: Fedora 33 -> 34, 34 -> 35, 35 -> 36

FEDORA_VERSION=36 dnf system-upgrade download -y --releasever=${FEDORA_VERSION} --exclude='kernel*'

# Reboot your Mac
dnf system-upgrade reboot

# After reboot clean old packages
dnf clean packages

## or clean all dnf cache
dnf clean all
```

## Not working

- Dynamic audio input/output change (on connecting/disconnecting headphones jack)
- TouchID - (@MCMrARM is working on it - https://github.com/Dunedan/mbp-2016-linux/issues/71#issuecomment-528545490)
- Microphone (it's recognised with new apple t2 sound driver, but there is a low mic volume amp)

## TODO

- add Fedora icon to usb installer
- alsa/pulseaudio config
  - Dynamic audio input/output change (on connecting/disconnecting headphones jack)

  ```bash
  ## to fix pipewire configs on current mbp-fedora installations
  sudo -i
  curl -Ls https://raw.githubusercontent.com/mikeeq/mbp-fedora/cdc2fa6e7ef53f995041f1b86d50f34587d7b738/files/audio/apple-t2.conf -o /usr/share/alsa-card-profile/mixer/profile-sets/apple-t2.conf
  curl -Ls https://raw.githubusercontent.com/mikeeq/mbp-fedora/cdc2fa6e7ef53f995041f1b86d50f34587d7b738/files/audio/91-pulseaudio-custom.rules -o /usr/lib/udev/rules.d/91-pulseaudio-custom.rules
  reboot

  ## if you're using MBP16,1 please apply those commands
  # https://gist.github.com/kevineinarsson/8e5e92664f97508277fefef1b8015fba
  sudo -i
  curl -Ls https://gist.github.com/kevineinarsson/8e5e92664f97508277fefef1b8015fba/raw/bb8319991923e1ad10f68f5c345706f2796ede21/AppleT2.conf -o /usr/share/alsa/cards/AppleT2.conf
  curl -Ls https://gist.github.com/kevineinarsson/8e5e92664f97508277fefef1b8015fba/raw/bb8319991923e1ad10f68f5c345706f2796ede21/apple-t2.conf -o /usr/share/alsa-card-profile/mixer/profile-sets/apple-t2.conf
  curl -Ls https://raw.githubusercontent.com/mikeeq/mbp-fedora/cdc2fa6e7ef53f995041f1b86d50f34587d7b738/files/audio/91-pulseaudio-custom.rules -o /usr/lib/udev/rules.d/91-pulseaudio-custom.rules
  reboot

  ## to manually change audio profile via PulseAudio cli execute
  # to headphones output
  pactl set-card-profile $(pactl list cards | grep -B10 "Apple T2 Audio" | head -n1 | cut -d "#" -f2) output:codec-output+input:codec-input

  # to speakers output
  pactl set-card-profile $(pactl list cards | grep -B10 "Apple T2 Audio" | head -n1 | cut -d "#" -f2) output:builtin-speaker+input:builtin-mic

  ## to manually adjust built-in mic volume
  pactl set-source-volume $(pactl list sources | grep -B3 "Built-in Mic" | head -n1 | cut -d"#" -f2) 300000
  ```

- disable iBridge network interface (awkward internal Ethernet device?)
- disable not working camera device
  - there are two video devices (web cameras) initialized/discovered, don't know why yet

    ```bash
    ➜ ls -l /sys/class/video4linux/
    total 0
    lrwxrwxrwx. 1 root root 0 Aug 23 15:14 video0 -> ../../devices/pci0000:00/0000:00:1d.4/0000:02:00.1/bce/bce/bce-vhci/usb7/7-2/7-2:1.0/video4linux/video0
    lrwxrwxrwx. 1 root root 0 Aug 23 15:14 video1 -> ../../devices/pci0000:00/0000:00:1d.4/0000:02:00.1/bce/bce/bce-vhci/usb7/7-2/7-2:1.0/video4linux/video1
    ➜ cat /sys/class/video4linux/*/dev
    81:0
    81:1
    ```

- verify `brcmf_chip_tcm_rambase` returns

## Known issues

- Kernel/Mac related issues are mentioned in kernel repo
- Anaconda sometimes could not finish installation process and it's freezing on `Network Configuration` step, probably due to iBridge internal network interface

> workaround - it's a final step of installation, just reboot your Mac (installation is completed)

- Macbooks with Apple T2 can't boot EFI binaries from HFS+ formatted ESP - only FAT32 (FAT32 have to be labelled as msftdata).

> workaround applied - HFS+ ESP is reformatted to FAT32 in post-scripts step and labelled as `msftdata`

- efibootmgr write command freezes Mac (it's executed in Anaconda during `Install bootloader...` step) - nvram is blocked from writing

  - since macOS Catalina EFI is blocked even from reading, so access to EFI is blocked via adding `efi=noruntime` to kernel args

    ```bash
    efibootmgr --c -w -L Fedora /d /dev/nvme0n1 -p 3 -l \EFI\fedora\shimx64.efi
    ```

- `ctrl+x` is not working in GRUB, so if you are trying to change kernel parameters - start your OS by clicking `ctrl+shift+f10` on external keyboard

## Docs

- Discord: <https://discord.gg/Uw56rqW>
- WiFi firmware: <https://packages.aunali1.com/apple/wifi-fw/18G2022>
- blog `Installing Fedora 31 on a 2018 Mac mini`: <https://linuxwit.ch/blog/2020/01/installing-fedora-on-mac-mini/>

### Fedora

- <https://fedoraproject.org/wiki/LiveOS_image>
- <https://docs.fedoraproject.org/en-US/quick-docs/creating-and-using-a-live-installation-image/>
- <https://pykickstart.readthedocs.io/en/latest/kickstart-docs.html#chapter-1-introduction>
- <https://forums.fedoraforum.org/showthread.php?309843-Fedora-24-livecd-creator-fails-to-create-initrd>
- <https://fedoraproject.org/wiki/QA/Test_Days/Live_Image>
- <https://fedoraproject.org/wiki/How_to_create_a_Fedora_install_ISO_for_testing>

### Github

- GitHub issue (RE history): <https://github.com/Dunedan/mbp-2016-linux/issues/71>
- VHCI+Sound driver (Apple T2): <https://github.com/MCMrARM/mbp2018-bridge-drv/>
- hid-apple keyboard backlight patch: <https://github.com/MCMrARM/mbp2018-etc>
- alsa/pulseaudio config files: <https://gist.github.com/MCMrARM/c357291e4e5c18894bea10665dcebffb>
- TouchBar driver: <https://github.com/roadrunner2/macbook12-spi-driver/tree/mbp15>
- Kernel patches (all are mentioned in github issue above): <https://github.com/aunali1/linux-mbp-arch>
- ArchLinux kernel patches: <https://github.com/ppaulweber/linux-mba>
- ArchLinux installation guide: <https://gist.github.com/TRPB/437f663b545d23cc8a2073253c774be3>
- hid-apple-patched module for changing mappings of ctrl, fn, option keys: <https://github.com/free5lot/hid-apple-patched>

## Credits

- @MCMrARM - thanks for all RE work
- @ozbenh - thanks for submitting NVME patch
- @roadrunner2 - thanks for SPI (touchbar) driver
- @aunali1 - thanks for ArchLinux Kernel CI
- @ppaulweber - thanks for keyboard and Macbook Air patches
