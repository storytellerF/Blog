
# YUMI（NSIS）软件界面模糊处理（用SetProcessDpiAwareness 使界面不再模糊）

## 探究

1. 下载YUMI的源码，解压打开，打开YUMI-Readme.txt 文件，下面是从中截取的片段
    >YUMI is an easy to use Multiboot script created using NSIS

    所以YUMI是用NSIS制作的。
2. 首先 了解NSIS是什么。这是一个用来快捷创建安装程序，一般安装程序的功能就是解压，粘贴，修改部分文件，添加注册表什么的（或者还有我不知道的😄)。所以什么安装程序，安装向导好多都是用NSIS 做的（虽然我在以前并没有了解过😄）。
3. 那么编辑YUMI的代码，然后重新编译就可以了。

## 步骤

1. 下载NSIS的编译器( [NSIS SOURCEFORGE](https://nsis.sourceforge.io/Download) ），在编译器中打开YUMI.nsi 这个文件，编译

    >!define: "TBM_SETPOS" already defined!

    很显然出错了

2. 由于这个宏已经被定义过了，导致编译失败然后停止（我不知道作者是如何编译通过的，可能是用了某个版本的编译器吧），现在开始找这个宏是在哪里声明或者定义的，然后就是我没有找到，反正解决方法有两个 一个是使用 ```!undef``` 将宏卸载或者删除 ，另一个是使用 ```!ifndef``` 和 ```!endif```(很类似C啊) 。如果对NSIS不了解可以通过NISI的软件查看用户手册（我之前也没有接触过NSIS，还有这个手册应该是个chm文件)

    ```c

    !undef TBM_SETPOS
    !define TBM_SETPOS 0x0405

    !undef TBM_GETPOS
    !define TBM_GETPOS 0x0400

    !undef TBM_SETRANGEMIN
    !define TBM_SETRANGEMIN 0x0407

    !undef TBM_SETRANGEMAX
    !define TBM_SETRANGEMAX 0x0408
    ```

    或者

    ```c
    !ifndef TBM_SETPOS
    !define TBM_SETPOS 0x0405
    !endif
    !ifndefTBM_GETPOS
    !define TBM_GETPOS 0x0400
    !endif
    !ifndefTBM_SETRANGEMIN
    !define TBM_SETRANGEMIN 0x0407
    !endif
    !ifndefTBM_SETRANGEMAX
    !define TBM_SETRANGEMAX 0x0408
    !endif
    ```

    这样编译就能够通过了
3. 开始做真正的代码修改了，找到 ```Function .onInit```的位置做如下修改

    ```c
    ; --- Stuff to do at startup of script ---
    Function .onInit
    ;在这里添加
    ```

    添加后是这样的

    ```c
    ; --- Stuff to do at startup of script ---
    Function .onInit
    System::Call 'SHCore::SetProcessDpiAwareness(i 2)i.R0'
    ```

    添加的内容是调用了一个Windows Api ，关于这个Api [SetProcessDpiAwareness](https://docs.microsoft.com/zh-cn/windows/desktop/api/shellscalingapi/nf-shellscalingapi-setprocessdpiawareness)，这里是我参考的博客 [NSIS对高分屏的支持以及解决方案](https://blog.csdn.net/eiilpux17/article/details/81676985)。
关于```System::Call``` 的有关信息可以到NSIS 的用户手册中查看

4. 再一次编译通过，然后测试，ok，结束（还有就是之所以模糊是因为Windows 的缩放导致的，特别是高分辨率的屏幕上，是默认进行缩放的，其实不缩放也很难用。图就不上了，反正也看不出当前界面是否进行了缩放，毕竟我是看到过通过修改注册表的方式修改缩放为100%来达到应用不模糊的聪明人。好像只要和注册表有关就很高级呢，什么问题都能解决呢 🤮）。
