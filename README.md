# Debian on Lenovo-x13s 
This outlines resources to get Debian running on Lenovo's x13s. Note, this process is far from stream lined. It is not for the novice Linux User. There is a high probability that you could ruin the windows install/laptop if you are not careful. To get things working well involves compiling custom kernels. 

## Features
Below are a list of features that "work". Please note that there is still a lot of development being done and even though things may "work" they may not be perfect.

- [x] USB
- [x] GUI Desktop (Gnome/Wayland)
- [x] Keyboard
- [x] Touchpad
- [ ] Touchscreen
- [x] WiFi (2.4 ghz & 5 ghz)
- [ ] Bluetooth
- [ ] LTE
- [ ] Accelerated Graphics
- [ ] Camera
- [x] Audio - not perfect but works

### Battery Life
I've noticed that currently, battery life is still much better in Windows 11 than it is in Debian. However, with the 6.5.y kernel and ASPM enabled, idle battery life is on par with Windows.

### Graphics
I have not confirmed the final state of graphics accerlation, but basic youtube videos on Firefox-esr works well enough. I could not get DRM in firefox-esr working so no Netflix/Amazon Prime, etc.

## Instalation
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

## IRC 

For more info on development, see irc on irc.oftc.net #aarch64-laptops. Please search [logs](https://oftc.irclog.whitequark.org/aarch64-laptops/2023-09-01) to help find answers quicker.

## Kernel Related stuff

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

### ASPM
Paraphrased from logs [here](https://oftc.irclog.whitequark.org/aarch64-laptops/2023-08-28) regarding ASPM which increases battery time 
*  idle time increases from from 10h to 15h
*  29h of suspend

To enable ASPM, pass `pcie_aspm.policy=powersupersave` on the kernel command line or enable it through sysfs 
