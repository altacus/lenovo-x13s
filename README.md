# lenovo-x13s

Debian Instalation directions
https://docs.google.com/document/d/1WuxE-42ZeOkKAft5FuUk6C2fonkQ8sqNZ56ZmZ49hGI/edit#heading=h.d1689esafsky

Follow all directions up to step 5. From step 5 follow the following steps

*  Configure package manager
*  Update the system
*  Update date and time
*  Restrict Wifi 5Ghz usage
*  upgrade scrypt - https://people.linaro.org/~manivannan.sadhasivam/x13s_upgrade/

This should allow you to get a decent enough working system without having to compile a new kernel.


kernel with working sound, wifi. 
https://github.com/steev/linux/tree/lenovo-x13s-linux-6.3.y

For more info on development, see irc on oftc #aarch64-laptops
https://oftc.irclog.whitequark.org/aarch64-laptops/2023-09-01

Using steev kernel 6.5.y to get audio working follow the follow steps (from [chat](https://oftc.irclog.whitequark.org/aarch64-laptops/2023-07-24))
*  Get the new [firmware](https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/commit/qcom/sc8280xp/LENOVO/21BX/audioreach-tplg.bin?id=f9a35b3f0779844aa686b76506344db70a72820d)
*  Apply top 4 patches [here](https://github.com/Srinivas-Kandagatla/alsa-ucm-conf/commits/x13s-volume-fixes)
*  Apply Patch [from](https://github.com/alsa-project/alsa-ucm-conf/commit/9bda3d15cc38bb705a1aa13f58adfea74bf37fe8)
*  Apply Patch [from](https://github.com/alsa-project/alsa-ucm-conf/pull/335)
