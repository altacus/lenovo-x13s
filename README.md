# lenovo-x13s

Debian Instalation directions [doc](https://docs.google.com/document/d/1WuxE-42ZeOkKAft5FuUk6C2fonkQ8sqNZ56ZmZ49hGI/edit#heading=h.d1689esafsky)

Follow all directions up to step 5. From step 5 follow the following steps

*  Configure package manager
*  Update the system
*  Update date and time
*  Restrict Wifi 5Ghz usage
*  upgrade [script](https://people.linaro.org/~manivannan.sadhasivam/x13s_upgrade/)

To get 5ghz wifi to work do the following
```
git clone https://github.com/kvalo/ath11k-firmware.git
cd ath11k-firmware
sudo cp -r WCN6855/hw2.0/* /lib/firmware/ath11k/WCN6855/hw2.0/
```


This should allow you to get a decent enough working system without having to compile a new kernel.

For more info on development, see irc on oftc #aarch64-laptops. Logs [here](https://oftc.irclog.whitequark.org/aarch64-laptops/2023-09-01)

Steev's Kernel with working sound, wifi, no additional patching necessary [6.3.y](https://github.com/steev/linux/tree/lenovo-x13s-linux-6.3.y)

Using Steev kernel [6.5.y](https://github.com/steev/linux/tree/lenovo-x13s-linux-6.5.y) to get audio working follow the follow steps (from [chat](https://oftc.irclog.whitequark.org/aarch64-laptops/2023-07-24))
*  Get the new [firmware](https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/commit/qcom/sc8280xp/LENOVO/21BX/audioreach-tplg.bin?id=f9a35b3f0779844aa686b76506344db70a72820d)
*  Apply top 4 patches [here](https://github.com/Srinivas-Kandagatla/alsa-ucm-conf/commits/x13s-volume-fixes)
*  Apply Patch [from](https://github.com/alsa-project/alsa-ucm-conf/commit/9bda3d15cc38bb705a1aa13f58adfea74bf37fe8)
*  Apply Patch [from](https://github.com/alsa-project/alsa-ucm-conf/pull/335)
