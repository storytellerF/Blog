# Clion 中自定义编译

CMake 不支持中文路径，所以自己做了一个GZMake，可以通过CMakeLists.txt 生成Makefile ，放到`gz_working` 目录。

地址 [gzmake](https://github.com/storytellerF/GZMake)

## 步骤

1. 首先添加`External Tools`

    File | Settings | Tools | External Tools

    ![gz make](https://upload-images.jianshu.io/upload_images/10647432-48aee387c1d6ce84.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 添加到工具栏（可以不做）

    ![添加到工具栏](https://upload-images.jianshu.io/upload_images/10647432-36341d76d7f2c92e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    需要选中`build`,然后再点击添加按钮，否则添加按钮是灰色的，也根本无法点击。

    还能够为其添加图标，如果不设置图标，显示的将会是默认图标。

    ![图标在这里](https://upload-images.jianshu.io/upload_images/10647432-fa5df1ebb54d1076.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    像一个纽扣。

3. 添加自定义配置

    ![自定义配置](https://upload-images.jianshu.io/upload_images/10647432-acd0519383c24681.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. 添加编译选项

    ![自定义](https://upload-images.jianshu.io/upload_images/10647432-343778d9bd347067.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    ![make](https://upload-images.jianshu.io/upload_images/10647432-d8200b05fce7a95f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    ![make clean](https://upload-images.jianshu.io/upload_images/10647432-50c61eaf02cac26a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于CMakeLists.txt 的改变，我们需要手动点击一下工具栏上的按钮。

经过配置，点击Run，会执行Executable的选项，再次之前会进行Build，build 会进行编译操作，编译完了便开始执行。因为我们使用的是make，如果代码没有发生改变，编译会立刻结束。

我们配置的make run，前面写着“Build”，其实就是下图的那个Build。

![Before Lanch](https://upload-images.jianshu.io/upload_images/10647432-6cae9f3487e05512.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后是jet brains 的关于这部分的介绍，如果有哪里不清楚，可以先查看这里[custom rundebug](https://www.jetbrains.com/help/clion/custom-build-targets.html#custom-rundebug)

最后，因为我们不再使用cmake，那么我们把cmake的功能关闭。

File | Settings | Build, Execution, Deployment | CMake

这个页面有一个"Enable profile"的选项，取消选中，关闭cmake 功能。甚至直接移除profile（不过放心，能移除，也是能够再添加回来的）。

Run没有问题，debug有问题，如果你的路径还有中文的话，想要debug只能到命令行手动调试。

原因是clion 在调试时会将断点信息输入到gdb中，然后我们的路径含有中文，gdb找不到相应的文件（编译后的exe文件），通过`dumpbin` 查看exe 的内容，发现中文路径是支持的，所以不支持的问题应该在gdb 或者某处的编码问题。然而自己写一个gdb 那可太麻烦了。
