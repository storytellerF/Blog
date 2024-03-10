# Clion 中自定义编译

CMake 不支持中文路径，所以自己做了一个GZMake，可以通过CMakeLists.txt 生成Makefile ，放到`gz_working` 目录。

地址 [gzmake](https://github.com/storytellerF/GZMake)

## 步骤

1. 首先添加`External Tools`

    File | Settings | Tools | External Tools

    ![gz make](https://i.bmp.ovh/imgs/2021/04/b7f1bc6f17f2b1ef.png)

2. 添加到工具栏（可以不做）

    ![添加到工具栏](https://i.bmp.ovh/imgs/2021/04/548e630449a22b23.png)

    需要选中`build`,然后再点击添加按钮，否则添加按钮是灰色的，也根本无法点击。

    还能够为其添加图标，如果不设置图标，显示的将会是默认图标。

    ![图标在这里](https://i.bmp.ovh/imgs/2021/04/e9733c7689ca4e6b.png)

    像一个纽扣。

3. 添加自定义配置

    ![自定义配置](https://i.bmp.ovh/imgs/2021/04/eab7f2b0409dfef6.png)

4. 添加编译选项

    ![自定义](https://i.bmp.ovh/imgs/2021/04/76cb9e9bbcf8ca14.png)

    ![make](https://i.bmp.ovh/imgs/2021/04/a901943e36fa563b.png)

    ![make clean](https://i.bmp.ovh/imgs/2021/04/f4ce811358bcb256.png)

对于CMakeLists.txt 的改变，我们需要手动点击一下工具栏上的按钮。

经过配置，点击Run，会执行Executable的选项，再次之前会进行Build，build 会进行编译操作，编译完了便开始执行。因为我们使用的是make，如果代码没有发生改变，编译会立刻结束。

我们配置的make run，前面写着“Build”，其实就是下图的那个Build。

![Before Lanch](https://i.bmp.ovh/imgs/2021/04/b26d0efab8361f9b.png)

最后是jet brains 的关于这部分的介绍，如果有哪里不清楚，可以先查看这里[custom rundebug](https://www.jetbrains.com/help/clion/custom-build-targets.html#custom-rundebug)
