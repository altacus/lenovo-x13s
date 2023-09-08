# lenovo-x13s Debian

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

General Kernel Building example (Steev) 
```
git clone https://github.com/steev/linux
git checkout lenovo-x13s-linux-6.5.y
make laptop_defconfig
make -j8 bindeb-pkg
```

note to append a local version to the kernel name 
```
make -j8 bindeb-pkg LOCALVERSION "-my123"
```

Steev's Kernel with working sound, wifi, no additional patching necessary for Debian [6.3.y](https://github.com/steev/linux/tree/lenovo-x13s-linux-6.3.y)

## 6.5.y Kernel 
There were issues encountered when using Steev kernel [6.5.y](https://github.com/steev/linux/tree/lenovo-x13s-linux-6.5.y). 

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
