# 面具

## 安装

如果您已经安装了Magisk，则 强烈建议 使用Magisk应用程序的“直接安装”方法直接进行升级。 以下教程仅适用于初始安装。
入门

在你开始之前：

    本教程假定您了解如何使用 adb 和 fastboot
    您设备的引导加载程序必须解锁
    在安装Magisk之前，请确保删除所有“引导映像模块”，例如其他根解决方案。 最简单的方法是使用工厂映像还原启动映像，或重新刷新 未预root的 自定义ROM
    如果您还打算安装自定义内核，请在Magisk之后安装

下载并安装最新的Magisk应用程序。 我们使用该应用程序收集有关您设备的一些信息。 在主屏幕中，您应该看到以下内容：

注意这个Ramdisk info。如果是yes 或者是‘是’，恭喜你，你的设备可以完美的安装Magisk。当然了，如果结果是no，意味着你的设备启动分区不包含ramdisk。这意味着你需要一些额外的步骤来让magisk正常工作。

    如果你的设备不包含ramdisk，阅读“安装”后面的“Recovery”章节。这里面的东西很重要。

如果你使用的是三星或者华为，而且“SAR” 的结果是yes，请阅读相关章节。

否则，直接去“Patching Images”章节。

(P.S.1 如果你的设备有ramdisk，你可以通过“Custom Recovery”来安装)（本人试验过了，失败）
(P.S.2 如果你对安卓是如何启动的，和magisk是如果起作用的，可以阅读这篇文章[https://topjohnwu.github.io/Magisk/boot.html])

## Patching Images

如果你的设备有ramdisk，你需要复制一份boot.img

如果你的设备没有ramdisk，你需要复制一份recovery.img。

你应该到官方“firmware”或者你的自定义rom 文件中提取这些文件（如果你用了的话），如果你还有问题，可以到 XDA-Developers （国外的一个论坛）查找资源，指导，讨论，或者在你的设备相关的板块寻求帮助。

    Copy the boot/recovery image to your device
    Press the Install button in the Magisk card
    If you are patching a recovery image, make sure “Recovery Mode” is checked in options.
    In most cases it should already be automatically checked.
    Choose “Select and Patch a File” in method, and select the stock boot/recovery image
    The Magisk app will patch the image to [Internal Storage]/Download/magisk_patched_[random_strings].img.
    Copy the patched image to your PC with ADB:
    adb pull /sdcard/Download/magisk_patched_[random_strings].img
    Flash the patched boot/recovery image to your device.
    For most devices, reboot into fastboot mode and flash with command:
    fastboot flash boot /path/to/magisk_patched.img or
    fastboot flash recovery /path/to/magisk_patched.img if flashing a recovery image
    Reboot and voila!（这部分，视频中演示了，不想翻译了）

## Custom Recovery

在一些自定义recovery （就是twrp，fox之类的）中，安装可能会失败的悄无声息（看起来成功了，但是失败了）。这是因为安装脚本无法正确的检测到你的设备信息，或者recovery 的环境不符合预期。如果你有这个问题，请使用“Patch Image” 里面的方法，这种方法百分百成功。正因如此，不推荐在新的设备上通过这种方式安装。这种方法只适合于一些“老旧”的设备。

    Download the Magisk APK
    Rename the .apk file extension to .zip, for example: Magisk-v22.0.apk → Magisk-v22.0.zip. If you have trouble renaming the file extension (like on Windows), use a file manager on Android or the one included in TWRP to rename the file.
    Flash the zip just like any other ordinary flashable zip.
    Warning: sepolicy.rule file of modules may be stored in cache partition, do not clear it.
    Check whether the Magisk app is installed. If it isn’t installed automatically, manually install the APK.（跟你安装第三方rom一样）

## Uninstallation/卸载

最好的卸载magisk 的方法就是直接卸载magisk 的软件。

如果你使用的是上面的“Custom recovery”，把app 安装文件重命名为“uninstall.zip”，然后就像刷rom 一样刷。

## Magisk in Recovery

如果你的设备没有ramdisk，没有办法，magisk，只能安装到recovery 分区。对于这种设备，每次你需要使用magisk 时都要重启到recovery。

当magisk 安装到recovery ，你无法通过“custom  recovery”安装或者升级magisk ，唯一的办法时通过magisk 的软件。软件可以获取你的手机的信息，安装到正确的分区，重启到正确的模式。

你可以通过组合键的方式让你进入recovery。

通过组合键，会首先进入magisk，此时不松手继续按住，会进入recovery。

## Samsung (System-as-root)

如果你的安卓版本不是9.0 或者更高，那就不用读了。

在安装面具之前

    安装面具会移除KNOX（不了解，请百度）
    第一次安装面具会请求格式化数据（不是指在解锁bootloader 时）。在继续下一步之前，备份你的数据。

### 解锁 Bootloader

在新款三星手机设备上解锁boot loader ，有一些的告诫：

    在开发者选项中允许OEM 解锁
    重启到 “download” 模式：手机关机，然后使用 “download模式”组合键。
    长按音量+ 来解锁bootloader。会清除你的数据然后自动重启。

你以为整好了，但其实没有。 Samsung 有 VaultKeeper, 意思是，bootloader 会拒绝所有的非官方的分区。除非VaultKeeper 明确允许。

    Go through the initial setup. Skip through all the steps since data will be wiped again later when we are installing Magisk. Connect the device to Internet during the setup.
    Enable developer options, and confirm that the OEM unlocking option exists and is grayed out. This means the VaultKeeper service has unleashed the bootloader.
    Your bootloader now accepts unofficial images in download mode

### Instructions

    Use either samfirm.js, Frija, or Samloader to download the latest firmware zip of your device directly from Samsung servers.
    Unzip the firmware and copy the AP tar file to your device. It is normally named as AP_[device_model_sw_ver].tar.md5
    Press the Install button in the Magisk card
    If your device does NOT have boot ramdisk, make sure “Recovery Mode” is checked in options.
    In most cases it should already be automatically checked.
    Choose “Select and Patch a File” in method, and select the AP tar file
    The Magisk app will patch the whole firmware file to [Internal Storage]/Download/magisk_patched_[random_strings].tar
    Copy the patched tar file to your PC with ADB:
    adb pull /sdcard/Download/magisk_patched_[random_strings].tar
    DO NOT USE MTP as it is known to corrupt large files.
    Reboot to download mode. Open Odin on your PC, and flash magisk_patched.tar as AP, together with BL, CP, and CSC (NOT HOME_CSC because we want to wipe data) from the original firmware. This could take a while (>10 mins).
    After Odin is done, your device should reboot. You may continue with standard initial setup.
    If you are stuck in a bootloop, agree to do a factory reset if promted.
    If your device does NOT have boot ramdisk, reboot to recovery now to boot Android with Magisk (reason stated in Magisk in Recovery).
    Although Magisk is installed, it still need some additional setup. Please connect to the Internet.
    Install the latest Magisk app and launch the app. It should show a dialog asking for additional setups. Let it do its job and the app will automatically reboot your device.
    Voila! Enjoy Magisk 😃

### Additional Notes

    Never, ever try to restore either boot or recovery partitions back to stock! You can easily brick your device by doing so, and the only way out is to do a full Odin restore with data wipe.
    To upgrade your device with a new firmware, NEVER directly use the stock AP tar file with reasons mentioned above. Always pre-patch AP in the Magisk app before flashing in Odin.
    Use HOME_CSC to preserve your data when doing a firmware upgrade in the future. Using CSC is only necessary for the initial Magisk installation.
    Never just flash only AP, or else Odin can shrink your /data filesystem. Flash full AP + BL + CP + HOME_CSC when upgrading.

## Huawei

面具不再支持华为的新机型，因为无法解bl锁。最重要的是华为不遵循安卓标准分区策略。The following are just some general guidance.

使用麒麟处理器的华为设备拥有不同分区方法。面具一般安装到启动分区，但是华为没有这个分区。

Generally, follow Patching Images with some differences from the original instructions:

    After downloading your firmware zip (you may find a lot in Huawei Firmware Database), you have to extract images from UPDATE.APP in the zip with Huawei Update Extractor (Windows only!)
    Regarding patching images:
        如果你的设备有ramdisk，使用ramdisk.img 而不是boot.img。
        如果你的设备没有ramdisk，使用recovery_ramdis.img（就长这样，不是输入错误）而不是recovery.img。
    When flashing the image back with fastboot
        If you patched RAMDISK.img, flash with command fastboot flash ramdisk magisk_patched.img
        If you patched RECOVERY_RAMDIS.img, flash with command fastboot flash recovery_ramdisk magisk_patched.img

This site is open source. Improve this page.
