# Linux on Lenovo-x13s 
This outlines resources to get Debian and Ubuntu running on Lenovo's x13s. This is not for the novice Linux User. There is a real chance that you could ruin the windows install/laptop if you are not careful. 

The Ubuntu installation is easier, but the scripts take longer to run. NOTE Ubuntu's booting process does not play nicely with bitlocker. The default kernel is 6.2. Upgrading from Lunar to Mantic will provide a 6.5 kernel (but read the fine print on upgrading). No kernel compilation/manual firmware updates are necessary.

The Debian installation is more involved. The default kernel is 6.0 but the provided script (on installation guide) will upgrade this to 6.2. The 6.4.1 kernel can be compiled and will work without additional firmware updates. Kernels later than 6.4.1 requires firmware/packages to be updated. **On Debian, grub package updates have been known to make the system unbootable.**

## Features
Below are a list of features that "work". Please note that there is still a lot of development being done and even though things may "work" they may not be perfect.

- [x] Accelerated Graphics - not perfect but works
- [x] Audio - not perfect but works
- [x] Bluetooth (Tested bluetooth mouse)
- [ ] Camera
- [x] Fingerprint sensor - 6.5 kernel or later
- [x] GUI Desktop (Gnome/Wayland)
- [x] Keyboard
- [ ] LTE
- [x] Touchpad
- [x] Touchscreen see [udev rule](https://github.com/altacus/lenovo-x13s#touchscreen)
- [x] WiFi (2.4 ghz & 5 ghz)
- [x] USB

### Battery Life
I've noticed that currently, battery life is still much better in Windows 11 than it is in Debian. However, with the 6.5.y kernel and ASPM enabled, idle battery life is on par with Windows.

### Graphics
I have not confirmed the final state of graphics accerlation, but basic youtube videos on Firefox-esr works well enough. I could not get DRM in firefox-esr working so no Netflix/Amazon Prime, etc.

# Ubuntu Installation

Ubuntu Installation are located ath the [X13s Concept page](https://launchpad.net/~ubuntu-concept/+archive/ubuntu/x13s) The directions there are pretty complete and thorough. In addition, upgrading to Mantic will provide the 6.5 Linux kernel, which is pretty up to date, without the need to upgrade firmware/compile kernel. This is the recommended distro for people who just want a working system with the least amount of effort.

# Debian Instalation
Debian Instalation directions [doc](https://docs.google.com/document/d/1WuxE-42ZeOkKAft5FuUk6C2fonkQ8sqNZ56ZmZ49hGI/edit#heading=h.d1689esafsky)

Follow all directions up to step 5. From step 5 follow the following steps

*  Configure package manager
*  Update the system
*  Update date and time
*  Restrict Wifi 5Ghz usage (if necessary)
*  upgrade [script](https://people.linaro.org/~manivannan.sadhasivam/x13s_upgrade/)

To get 5ghz wifi to work, upgrade the ath11k firmware
```
git clone https://github.com/kvalo/ath11k-firmware.git
cd ath11k-firmware
sudo cp -r WCN6855/hw2.0/* /lib/firmware/ath11k/WCN6855/hw2.0/
```
This should allow you to get a decent enough working system without having to compile a new kernel.

# IRC 

For more info on development, see irc on irc.oftc.net #aarch64-laptops. Please search [logs](https://oftc.irclog.whitequark.org/aarch64-laptops/2023-09-01) to help find answers quicker.

# Kernel and Firmware Related stuff

Install pre-requisites

```shell
sudo apt install bc bison build-essential debhelper flex libssl-dev make rsync
```

General Kernel Building example (Steev) 
```shell
git clone https://github.com/steev/linux
git checkout lenovo-x13s-linux-6.5.y
make laptop_defconfig
make -j8 bindeb-pkg
```

To append a local version to the kernel name 
```shell
make -j8 bindeb-pkg LOCALVERSION "-my123"
```

The debian packages will be in the parent directory.  To install

```shell
cd ..
dpkg -i linux-image-6.0.0-rc3+_6.0.0-rc3+-3_arm64.deb
```

The DTB file which is used during the boot process is located in /boot/efi/dtb folder. Hence after installing a new kernel it is recommended to update the DTB file with the new version. The DTB files are different for the Lenovo X13s and the Makena CRD:

To update the DTB, simply override them, e.g.:

```shell
sudo cp /usr/lib/<kernel version>/qcom/sc8280xp-lenovo-thinkpad-x13s.dtb /boot/efi/dtb/f249803d-0d95-54f3-a28f-f26c14a03f3b.dtb
```

Steev's Kernel with working sound, wifi, no additional patching necessary for Debian [6.3.y](https://github.com/steev/linux/tree/lenovo-x13s-linux-6.3.y)

## Firmware
If you are missing firmware, a good resource would be this [repo.](https://github.com/ironrobin/x13s-alarm) After updating the firmware, re-run update initramfs to be safe.

```
sudo update-initramfs -u
```

## 6.5.y Kernel 
Steev's [6.5.y](https://github.com/steev/linux/tree/lenovo-x13s-linux-6.5.y) has significant improvements, however, at the time of this writing, the Debian packages in sid have not caught up to the kernel development. To resolve the Audio issues continue below.

### Audio issues resolution (dummy output only)
To get audio working follow the steps (from [chat](https://oftc.irclog.whitequark.org/aarch64-laptops/2023-07-24)) I included links here for thoroughness.
*  Get the new [firmware](https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/commit/qcom/sc8280xp/LENOVO/21BX/audioreach-tplg.bin?id=f9a35b3f0779844aa686b76506344db70a72820d)
*  Apply top 4 patches by [Srinivas-Kandagatla](https://github.com/Srinivas-Kandagatla/alsa-ucm-conf/commits/x13s-volume-fixes)
*  Apply Device Number [Patch](https://github.com/alsa-project/alsa-ucm-conf/commit/9bda3d15cc38bb705a1aa13f58adfea74bf37fe8)
*  Apply Volume Fix [Patch](https://github.com/alsa-project/alsa-ucm-conf/pull/335)

The changes are reflected in files in the [/lib](https://github.com/altacus/lenovo-x13s/tree/main/lib) and [/usr](https://github.com/altacus/lenovo-x13s/tree/main/usr) directories.

## ASPM
Paraphrased from logs [here](https://oftc.irclog.whitequark.org/aarch64-laptops/2023-08-28) regarding ASPM which increases battery time 
*  idle time increases from from 10h to 15h
*  29h of suspend

To enable ASPM, pass `pcie_aspm.policy=powersupersave` on the kernel command line or enable it through sysfs 

## Touchscreen

Tested with 6.5.y kernel (and later). To get the touchscreen working, copy the [72-x13s-touchscreen.rules](https://github.com/ironrobin/x13s-alarm/blob/trunk/x13s-touchscreen-udev/72-x13s-touchscreen.rules) file to `/lib/udev/rules.d/` and reboot. The text of the file is below.

72-x13s-touchscreen.rules
```
ACTION=="add" \
, RUN+="/bin/bash -c 'echo 4-0010 > /sys/bus/i2c/drivers/i2c_hid_of/bind'"
```

