# VMware Workstation 虚拟机无法更新VMware Tools（由于没有光驱来挂载镜像）

## 问题研究

VMware Workstation 中的VMware Tools 是通过光驱安装的，在第一次安装系统时，添加了一个CD/DVD驱动器的配置，并为他添加了一个iso的系统镜像，系统安装完成后，安装VMware Tools，VMware 会把`VMware\VMware Workstation\windows.iso` 的文件挂载到这个CD/DVD驱动器中，当我把这些全部安装完成时就删除了这个CD/DVD。

上面说的本来也没有什么问题的，只是在我升级VMware Workstation 后，会提示更新这个VMware Tools，此时是没有CD/DVD驱动器的，`windows.iso` 自然也无法挂载，自然也无法更新。

想要解决这个问题，那就需要添加一个CD/DVD驱动器。

## 解决

1. 选择编辑虚拟机配置，添加CD/DVD 驱动器。（如果想要编辑虚拟机配置，需要将虚拟机关机）。
![虚拟机配置](https://upload-images.jianshu.io/upload_images/10647432-e0d920bf5254c6cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击完成
2. 已经添加完成，还要继续对这个CD/DVD驱动器进行设置。

    - 选择使用物理驱动器，因为不需要自己挂载镜像文件。
    - 勾掉启动时连接，因为没有挂载镜像文件，也没有真正的光驱设备，导致启动时找不到这个设备，会显示“无法连接，以后还要不要自动连接”的提示，我们这里将他勾掉，勾掉之后他就不会自动连接，自然也没有这个提示。不过，不管勾不勾都不影响后面的使用。

    ![虚拟机配置](https://upload-images.jianshu.io/upload_images/10647432-39bf8fda397c968e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 将虚拟机开机，选择菜单中的 虚拟机->更新VMware Tools，就可以正常更新VMware Tools 了。
